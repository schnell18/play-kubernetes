# Introduction

Nitty-gritty of deploying a simple application.

## Deploy via dashboard

You have three options to create application via dashboard:
- Create from input
- Create from file
- Create from form

The first options allows you enter the json/yaml representation of the application in
question. The second option is similar, you specify the json/yaml using a file.
With the third option, you specify key attributes such as:
- application name
- image name
- number of pods
- service

You can set more options by enabling `show advanced options`.
The option creates a deployment under hood.

## Deploy via CLI

You may using kubectl to deploy an appliction. Here is an example to deploy
nginx:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: webserver
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.19.6-alpine
            ports:
            - containerPort: 80

Then you expose the web server with two methods.
The first method creates the service by using a yaml file:

  apiVersion: v1
  kind: Service
  metadata:
    name: web-service
    labels:
      run: web-service
  spec:
    type: NodePort
    ports:
    - port: 80
      protocol: TCP
    selector:
      app: nginx

The second method works only if a deployment exists for the underlying service.
You use `kubectl expose` to expose the nginx in this example by using:

  kubectl expose deployment webserver --name=web-service --type=NodePort

You can get the node port by typing command as follows:

  kubectl get services

  NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
  kubernetes    ClusterIP   10.96.0.1      <none>        443/TCP        2d16h
  web-service   NodePort    10.107.55.66   <none>        80:30048/TCP   13m

Or you can use `kubectl describe` command if you know the name of service:

  kubectl describe service web-service

  Name:                     web-service
  Namespace:                default
  Labels:                   app=nginx
                            run=web-service
  Annotations:              <none>
  Selector:                 app=nginx
  Type:                     NodePort
  IP:                       10.107.55.66
  Port:                     <unset>  80/TCP
  TargetPort:               80/TCP
  NodePort:                 <unset>  30048/TCP
  Endpoints:                172.17.0.6:80,172.17.0.7:80,172.17.0.8:80
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>

Search for `NodePort` for the port you will connect the browser to. In this case,
it is 30048. You need get the IP address of the node. In the case of minikube,
you type:

  minikube ip

  192.168.99.104

It is `192.168.99.104` in this case. So you connect your browser to:

  http://192.168.99.104:30048/

To access the default nginx web page.

You may access the web page in one go if you are using minikube:

  minikube service web-service

  |-----------|-------------|-------------|-----------------------------|
  | NAMESPACE |    NAME     | TARGET PORT |             URL             |
  |-----------|-------------|-------------|-----------------------------|
  | default   | web-service |          80 | http://192.168.99.104:30048 |
  |-----------|-------------|-------------|-----------------------------|
  🎉  正通过默认浏览器打开服务 default/web-service...

You may substitute `web-service` with your service name as you see fit.

## Liveness probe

Liveness means application in the container responds to request as desgined.
Kubernetes will restart container if liveness probe is failed.
To specify liveness probe, you have three options:
- Liveness command
- Liveness HTTP request
- TCP liveness probe

Liveness probe using command example:

  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      test: liveness
    name: liveness-exec
  spec:
    containers:
    - name: liveness
      image: busybox:1.32.1
      args:
      - /bin/sh
      - -c
      - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
      livenessProbe:
        exec:
          command:
          - cat
          - /tmp/healthy
        initialDelaySeconds: 3
        failureThreshold: 1
        periodSeconds: 5

If command exits with code 0, then the pod is healthy. Otherwise, it is
unhealthy and kubernetes will attempt to restart it.

Liveness probe using http example:

  ...
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
          httpHeaders:
          - name: X-Custom-Header
            value: Awesome
        initialDelaySeconds: 3
        periodSeconds: 3
  ...

If the probe returns http code 2xx, then the pod is healthy. Otherwise, it is
unhealthy and kubernetes will attempt to restart it.

Liveness probe using tcp example:

  ...
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
  ...

Kubernetes try to open a TCP connection to the pod on the specified pod. If the
connection is established successfully, then the pod is healthy. Otherwise, it
is unhealthy and kubernetes will attempt to restart it.

## Readiness probe

Readiness probe is to check whether the pod in question is ready to serve
external requests. Readiness may be different from liveness when the
containerized application request extra setup such as loading of big dataset,
startup of dependent services etc. The configuration of readiness probe is the
same as liveness probe.
