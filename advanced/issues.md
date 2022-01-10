# minikube fail to enable ingress

## problem statement

Run `minikube addons enable ingress` generates errors like:

    🔎  Verifying ingress addon...

    ❌  Exiting due to MK_ENABLE: run callbacks: running callbacks: [waiting for app.kubernetes.io/name=ingress-nginx pods: timed out waiting for the condition]

    😿  If the above advice does not help, please let us know:
    👉  https://github.com/kubernetes/minikube/issues/new/choose


## Cause analysis

The problem is caused by the two images not being pulled successfully:

- registry.cn-hangzhou.aliyuncs.com/google_containers/controller:v0.40.2
- registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.2.2

The long repository is caused by setting minikube's `image-repoistory` option.
In fact, the `regristry.cn-hangzhou.aliyuncs.com` can be shortened as
`registry.aliyuncs.com`.
The two images are absent from regristry of `aliyun`. According to [source code
of the offical nginx ingress controller][1], the second image is hosted on
dockerhub rather than k8s.gcr.io. This seems to be a mistake made by minikube.

## Solution

For the first image, pull the `k8s.gcr.io/ingress-nginx/controller:v0.40.2`
image and use `docker save` to save it into a file, then transport to the
worker node and use `docker load` to load the image. Finally, change the
repository prefix to `k8s.gcr.io/ingress-nginx`.

For the second image, change the repository prefix to `docker.io/jettech`.
Then you should be able to pull the image successfully.


[1]: https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.40.2/deploy/static/provider/baremetal/deploy.yaml
