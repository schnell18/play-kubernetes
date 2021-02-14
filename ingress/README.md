# Introduction

Nitty-gritty of ingress.

## Reason to have ingress

Centralize rules to route inbound connection to cluster services.
Features of ingress include:

- TLS (Transport Layer Security)
- Name-based virtual hosting
- Fanout routing
- Loadbalancing
- Custom rules

Ingress only defines the rules, you need ingress controller to enforce the
rule.  An ingress controller is an applicatioin watching the master node's
ingress resource changes and make updates to load balancer accordingly. Ingress
controller is also known as, controler, ingress proxy, service proxy or reverse
proxy. Kubernetes supports ingress controllers such as:

- nginx ingress controller
- contour
- HAProxy ingress
- Istio
- Kong
- Traefik
- GCE L7 Load Balancer Controller

## explore ingress

### create blue/green application

The Dockerfile for blue/green apps are located under `apps` directory.
You may build and push the image to dockerhub.

### deploy blue/green to kubernetes

Here is the service yaml file for blue/green apps:

    apiVersion: v1
    kind: Service
    metadata:
      name: blueapp-svc
    spec:
      type: NodePort
      selector:
        app: blueapp
      ports:
      - protocol: TCP
        port: 80
        targetPort: 80

    apiVersion: v1
    kind: Service
    metadata:
      name: greenapp-svc
    spec:
      type: NodePort
      selector:
        app: greenapp
      ports:
      - protocol: TCP
        port: 80
        targetPort: 80

Here is the deployment yaml file for blue/green apps:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: blueapp
      labels:
        app: blueapp
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: blueapp
      template:
        metadata:
          labels:
            app: blueapp
        spec:
          containers:
          - name: blueapp
            image: schnell18/blueapp:0.1
            ports:
            - containerPort: 80

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: greenapp
      labels:
        app: greenapp
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: greenapp
      template:
        metadata:
          labels:
            app: greenapp
        spec:
          containers:
          - name: greenapp
            image: schnell18/greenapp:0.1
            ports:
            - containerPort: 80

### enable nginx ingress controller

Minikube bundles nginx ingress controller. However, it is disabled by default.
To enable it, you type:

    minikube addons enable ingress

### define ingress object

To route blue/green apps with ingress, you create an ingress object as follows:

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ingress-demo
      namespace: default
    spec:
      rules:
      - host: blue.example.com
        http:
          paths:
          - backend:
              service:
                name: blueapp-svc
                port:
                  number: 80
            path: /
            pathType: ImplementationSpecific
      - host: green.example.com
        http:
          paths:
          - backend:
              service:
                name: greenapp-svc
                port:
                  number: 80
            path: /
            pathType: ImplementationSpecific

You may delete the ingress validation should there is any error saying
validation failed:

    kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

This not the best solution but installation will pass smoothly.

Review the `ingress-demo` by typing:

    kubectl describe ingress ingress-demo

    Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
    Name:             ingress-demo
    Namespace:        default
    Address:
    Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
    Rules:
      Host               Path  Backends
      ----               ----  --------
      blue.example.com
                         /   blueapp-svc:80 (172.17.0.16:80,172.17.0.17:80)
      green.example.com
                         /   greenapp-svc:80 (172.17.0.18:80,172.17.0.19:80)
    Annotations:         <none>
    Events:              <none>

Change `/etc/hosts` to map `blue.example.com` and `green.example.com` to the IP
address of minikube master node.  Open http://blue.example.com and you will see
web page w/ blue background.  Open http://green.example.com and you will see
web page w/ green background.
