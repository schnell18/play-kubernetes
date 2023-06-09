Steps for Replacing Amazon VPC CNI Label Existing Nodes At first, all running
nodes in the EKS cluster should be labelled with cni-plugin=aws in order to
constrain the aws-node Daemonset only runs on these nodes on the next step.

Here is my script to adding the label for all nodes in a loop with checking if the label already being added.

    for NODE in $(kubectl get node --output=jsonpath={.items..metadata.name}); do
        LABELLED=$(kubectl get node $NODE -o json | jq '.metadata.labels | has("cni-plugin")')
        if [ "$LABELLED" = "true" ]; then
            LABEL_VALUE=$(kubectl get node $NODE -o json | jq -r '.metadata.labels.cni-plugin')
            echo "Node $NODE already labelled: cni-plugin=$LABEL_VALUE"
        else
            kubectl label nodes $NODE cni-plugin=aws
        fi
    done

After running the script, it can be verified with following commands that all nodes are labelled correctly.

    # verify all nodes are labelled correctly
    LABELLED_NODE_COUNT=$(kubectl get node -l cni-plugin=aws -o json | jq '.items | length')
    ALL_NODE_COUNT=$(kubectl get node -o json | jq '.items | length')

    if [ ! "$LABELLED_NODE_COUNT" = "$ALL_NODE_COUNT" ]; then
        echo "Not all nodes labelled."
        echo "Labelled nodes: $LABELLED_NODE_COUNT, total: $ALL_NODE_COUNT"
        exit 1
    fi

    echo "All nodes are labelled."

## Patch aws-node Daemonset
The nodeAffinity field in the Daemonset can be used
for specifying which node the Daemonset Pods can run on, so the aws-node
    Daemonset can be patched as:

    kubectl patch daemonset aws-node -n kube-system --patch "$(cat aws-node-patch.yaml)"
Here is the content of aws-node-patch.yaml:

    spec:
      template:
        spec:
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      # This is to limit aws-node only running on the nodes
                      # with label cni-plugin=aws
                      - key: cni-plugin
                        operator: In
                        values:
                          - aws
                      # These below are what aws-node already has
                      - key: kubernetes.io/os
                        operator: In
                        values:
                          - linux
                      - key: kubernetes.io/arch
                        operator: In
                        values:
                          - amd64
                          - arm64
                      - key: eks.amazonaws.com/compute-type
                        operator: NotIn
                        values:
                          - fargate

## Install Cilium CNI

With replacing Amazon VPC CNI, Cilium CNI needs to do the similar jobs that VPC
CNI does for allocating AWS ENI IP addresses for each pods, so it needs to set
eni.enabled=true and tunnel=disabled.

Meanwhile, the Daemonset of Cilium CNI needs to run on the node without the
label cni-plugin=aws on the contrary to Amazon VPC CNI.

    helm install cilium cilium/cilium \
        --version 1.12.4 \
        --namespace kube-system \
        -f cilium-values.yaml

Here is the contents of cilium-value.yaml.

    eni:
      enabled: true
    ipam:
      mode: eni
    egressMasqueradeInterfaces: eth0
    tunnel: disabled
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            # This is to limit cilium only running on the nodes
            # WITHOUT label cni-plugin=aws
            - key: cni-plugin
              operator: NotIn
              values:
              - aws

For more installing options, please refer to https://docs.cilium.io/en/v1.12/gettingstarted/k8s-install-helm/

After running the helm install, the cilium command can be used to check the installation status.

cilium status --wait

And the pod count of cilium Daemonset is supposed to be 0, but it might be
other values if a new node is launched in the meantime. So I use following
commands to double check:

    # validate cilium daemonset only runs on unlabelled nodes
    DESIRED_COUNT=$(kubectl get daemonset cilium -n kube-system -o json | jq '.status.desiredNumberScheduled')
    UNLABELLED_NODE_COUNT=$(kubectl get nodes -l '!cni-plugin' -o json | jq '.items | length')

    if [ ! "$DESIRED_COUNT" = "$UNLABELLED_NODE_COUNT" ]; then
        echo "cilium daemonset should ONLY run on unlabelled nodes."
    fi

## Refresh Nodes
Now, the nodes with the label cni-plugin=aws can be deleted one by one and the
Pods would be rescheduled automatically.

For removing a node from a Kubernetes cluster safely, each node with the label
cni-plugin=aws needs to be cordoned, drained and deleted with kubectl and then
terminated from the auto scaling group with following commands.

    NODE="<node_name>"
    REGION="<EKS_cluster_region>"

    echo "cordon the node: $NODE"
    kubectl cordon $NODE

    echo "drain the node: $NODE"
    kubectl drain $NODE --delete-local-data --ignore-daemonsets --force

    echo "remove the node from EKS cluster: $NODE"
    kubectl delete node $NODE

    NODE_ID=$(aws ec2 describe-instances \
        --region=$REGION \
        --filter Name=private-dns-name,Values=$NODE --no-cli-pager \
        | jq -r ".Reservations[0].Instances[0].InstanceId")

    echo "take the node out of autoscaling group: $NODE ($NODE_ID)"
    aws autoscaling terminate-instance-in-auto-scaling-group \
        --region=$REGION \
        --instance-id $NODE_ID \
        --should-decrement-desired-capacity \
        --no-cli-pager

Several things need to be verified in the process of refreshing nodes:

- Check the pod count of aws-node Daemonset get decreased after deleting a labelled node;
- Check the pod count of cilium Daemonset get increased if a new node is launched and added to the cluster;
- Run cilium status to check the status of cilium Daemonset, cilium-operator, cluster pods, etc;
- Check the status of application Pods rescheduled to the new nodes;
- Delete aws-node Daemonset

When all lablled node get deleted from the EKS cluster, the Amazon VPC CNI
(aws-node Daemonset) is good to be deleted as well.

    # validate aws-node daemonset has 0 pods

    DESIRED_COUNT=$(kubectl get daemonset aws-node -n kube-system -o json | jq '.status.desiredNumberScheduled')

    if [ ! "$DESIRED_COUNT" = "0" ]; then
        echo "aws-node still runs on some nodes, please referesh all nodes before deleting it."
        exit 1
    fi

    echo "No pod of aws-node is running, safely delete it."
    kubectl delete daemonset aws-node -n kube-system

As Amazon VPC CNI is an add-on of the EKS cluster, now it can be removed from AWS console as well.

After completing all the migration steps, the EKS cluster runs on top of Cilium
CNI only and all Cilium features are in hands.
