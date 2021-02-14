# Introduction

Various topics on quota, autoscaling, jobs, daemonset, statefulset, crd,
federation, network police, monitoring/loggin, helm and service mesh.

## annotation

Non-identifying key-value pair to store meta-info kubernetes object.
Appropriate information to record as annotation include:

- build/release IDs, PR number, git commit IDs
- pointers to logging, monitoring, debugging tools etc

Sample annotation usage:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: webserver
      annotations:
        description: Deployment based PoC dates 2nd May'2019

## Quota and limits

Quota and limit enable fair resource allocation to cluster users.
Supported quota includes:
- Compute Resource quota
- Storage Resource quota
- Object Count quota

The [LimitRange][1] allows fine-grained resource usage control.

## Autoscaling

Automatically resize computing nodes based on requests.
Kubernetes support three types of autoscaling:
- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- Cluster Autoscaler

## Jobs and CronJobs

Job support batch processing. A job creates one or more pods to perform actual
work. It ensures the completion of job. You can configure job by the following
parameters:

- parallelism
- completions
- activeDeadlineSeconds
- backoffLimit
- ttlSecondsAfterFinished

CronJob (requires kubernetes 1.4+) support extra parameters:

- startingDeadlineSeconds
- concurrencyPolicy

## Daemonset

Daemonset manages pods running on each and every node with exact one instance.
kube-proxy, log collector are good examples of daemonset.

## StatefulSet

StatefulSet manages pods running in strict order, or requires unique identity.
MySQL cluster, etcd cluster are good examples of statefulset.

## CRD

CRD stands for custom resource definition. It is underlying model for
kubernetes operator pattern. CRD requires corresponding custom controller, which
interpret the CRD and perform the request actions.

## Kubernetes federation

Kubernetes federation enable us to manage multiple cluster using one control
plane. It serves a multi-datacenter failover solution.

## Network policy

Network policy defines how pods are allowed to communicate to pods and resource
inside or outside cluster.

## Monitoring and logging

Kubernetes needs collect resource usage data by pod, service, node etc.
Two popular monitoring solutions include:
- Metrics Server
- Prometheus

Logging requires third-party tools. Fluentd and Elasticsearch are very common
in kubernetes community.

## Helm

Helm is a tool to simplify complex application deployment in kubernetes.
Key concepts in helm include:
- chart
- release
- repository

Helm is like a package manager of application running on kubernetes.

## Service Mesh

Service Mesh is a third party solution to kubernetes native application.  It
include control plane and data plane. The Control Plane runs agents responsible
for the service discovery, telemetry, load balancing, network policy, and
gateway. The Data Plane proxy component is typically injected into Pods, and it
is responsible for handling all Pod-to-Pod communication, while maintaining a
constant communication with the Control Plane of the Service Mesh.

Popular service mesh implementations include:
- Consul
- Envoy
- Istio
- Kuma
- Linkerd
- Maesh
- Tanzu Service Mesh

[1]: https://kubernetes.io/docs/concepts/policy/limit-range/
