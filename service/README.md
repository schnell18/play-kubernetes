# Introduction

Nitty-gritty of kubernetes service.

## reason to have service

Pod is ephemeral and client should not use IP address of pod
to locate the service it provides. As a result, kubernetes makes
an abstraction called `service` to group multiple pods providing
same functions. Request to `service` is routed to the underlying
pod by kubernetes.

## Service types

Kuberenetes supports the following service types:
- ClusterIP: accessible only from within the cluster
- NodeIP: use high port within 30000-32767 on node to access service
- LoadBalancer: ClusterIP and NodeIP are created automatically,
                external load balancer routes request to one of nodes
- ExternalIP:
- ExternalName: returns CNAME record of external service

## combine pods to make service

Kuberenetes use label and selector to group pods as service.
Service has virtual IP address.
Besides pod, kubernetes can also expose ReplicaSet, DaemonSet and
StatefulSet as service.

Here is an example of service object to expose HTTP service:

    apiVersion: v1
    kind: Service
    metadata:
      name: frontend-svc
    spec:
      selector:
        app: frontend
      ports:
      - protocol: TCP
        port: 80
        targetPort: 5000

The combination of pod IP address and targetPort is referred to as
service endpoint. Service endpoint is managed by kubernetes
automatically, which is the major task of `kubeproxy'.

## kube-proxy

Watch apiserver change and change iptable rules accordingly.

## Service discovery

Kuberenetes support two service discovery methods:

- environment variable
- DNS

Environment variables may not set before properly when a pod
is started.

Say, you have a service named `redis-master`, you can access the
service by using environment variable `REDIS_MASTER_SERVICE_HOST`.

You can also use `redis-master` as hostname within the same namespace. The
general form of the FQDN is something like:

    <service-name>.<namespace>.svc.<cluster-name>

