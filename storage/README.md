# Introduction

Nitty-gritty of persistent volume and persistent volume claim.

## Reason to have persistent volume

By default, container uses layered file system, any the file system and data go
away when the container is stopped.  Some stateful application requires that
data be persistent across container's life span. To solve this problem,
kubernetes provides persistent volume to allow application store data without
worrying of data loss. The types of volume supported by kubernetes are:

- emptyDir
- hostPath
- nfs
- iscsi
- secret
- configMap
- cephfs
- persistentVolumeClaim
- gcePersistentDisk
- awsElasticBlockStore
- azureDisk
- azureFile

PersistentVolume is the abstraction to manage storage, while
PersistentVolumeClaim is the abstraction to consume storage.

PersistentVolume can be created manually by administrator or automatically via
`storageClass` mechanism.
Volume types support management through persistent volume are:

- CephFS
- NFS
- GCEPersistentDisk
- AWSElasticBlockStore
- AzureDisk
- AzureFile
- iSCSI

## persistent volume claim

A PersistentVolumeClaim (PVC) is a request for storage by a user. Users request
storage based on type, access mode and size. The access mode consists of:

- ReadWriteOnce
- ReadOnlyMany
- ReadWriteMany

## Container Storage Interface (CSI)

It is a standardized interface to work with various container orchestrators and
storages. CSI suppport in kubernetes is stable now(as of v1.13).

## hostPath example

This is a simple example to use `hostPath` volume.

    apiVersion: v1
    kind: Pod
    metadata:
      name: share-pod
      labels:
        app: share-pod
    spec:
      volumes:
      - name: host-volume
        hostPath:
          path: /home/docker/pod-volume
      containers:
      - image: nginx:1.19.6-alpine
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: host-volume
      - image: debian:10.8
        name: debian
        volumeMounts:
        - mountPath: /host-vol
          name: host-volume
        command: ["/bin/sh", "-c", "echo Introduction to Kubernetes > /host-vol/index.html; sleep 3600"]

This example deploys a nginx web server and set its document root to a
directory managed by `hostPath`. Then it creates a debian container to change
the `index.html` to `Introduction to Kubernetes`.

To check the result web page, you may expose the pod as a service as:

    kubectl expose pod share-pod --type=NodePort --port=80

Then you locate the random node port and node IP by:

    kubectl get services
    minikube ip

With these information, you can open a browse to see the nginx home page.
You can also do this in one go using:

    minikube service share-pod

Now you delete the `share-pod` and check what happens to `share-pod` service:

    kubectl get svc,endpoints
    NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    service/kubernetes    ClusterIP   10.96.0.1      <none>        443/TCP        2d20h
    service/share-pod     NodePort    10.104.23.50   <none>        80:30807/TCP   16m
    service/web-service   NodePort    10.107.55.66   <none>        80:30048/TCP   3h41m

    NAME                    ENDPOINTS                                   AGE
    endpoints/kubernetes    192.168.99.104:8443                         2d20h
    endpoints/share-pod     <none>                                      16m
    endpoints/web-service   172.17.0.6:80,172.17.0.7:80,172.17.0.8:80   3h41m

As you can see, the `share-pod` remains, however, there is no endpoint to serve
this service.  Now create another nginx to use the original `hostPath` volume.

    apiVersion: v1
    kind: Pod
    metadata:
      name: check-pod
      labels:
        app: share-pod
    spec:
      volumes:
      - name: host-volume
        hostPath:
          path: /home/docker/pod-volume
      containers:
      - image: nginx:1.19.6-alpine
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: host-volume

