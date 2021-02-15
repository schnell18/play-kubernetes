# Introduction

Nitty-gritty of minikube installation and usage.

## Verify virtualization setting

Before you install and run minikube, you need make sure the host environment
support virtualization properly. On Linux you may run the command that follows:

    # Verify the virtualization support on your Linux OS
    # (a non-empty output indicates supported virtualization):
    grep -E --color 'vmx|svm' /proc/cpuinfo

On Mac OSX, you type the following command instead:

    # Verify the virtualization support on your macOS
    # (VMX in the output indicates enabled virtualization)
    sysctl -a | grep -E --color 'machdep.cpu.features|VMX'

## Installation

On Arch/Manjaro, you type:

    sudo pacman -S minikube

On Mac OSX, you type:

    wget https://github.com/kubernetes/minikube/releases/download/v1.17.1/minikube-darwin-amd64


## Bootstrap a kubernetes cluster

You run:

    minikube start \
        --kubernetes-version=v1.20.2 \
        --image-mirror-country=cn\
        --image-repository=registry.aliyuncs.com/google_containers \
        --driver=virtualbox

Substitute the kubernetes version as you see fit.

## Launch dashboard

You run:

    minikube dashboard

and minikube will launch a browser and bring you to the dashboard webui.

## Access virtual host


You run:

    minikube ssh

to access the remote console of the single node virtual host runs k8s cluster.

## Access kubernetes using API

With `kubectl proxy` running, you may access kuberenets APIs using:

	curl http://localhost:8001/
	curl http://localhost:8001/api/v1
	curl http://localhost:8001/apis/apps/v1
	curl http://localhost:8001/healthz
	curl http://localhost:8001/livez
	curl http://localhost:8001/logs
	curl http://localhost:8001/metrics

Without `kubectl proxy` running, you may access kuberenets APIs using:

- Bearer Token
- Certificates

Using Bearer Token approach:

    TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d ' ')
    APISERVER=$(kubectl config view | grep https | grep 8443 | cut -f2- -d ':' | tr -d ' ')
	curl -s $APISERVER/version --header "Authorization: Bearer $TOKEN" --insecure

Using certificate

    APISERVER=$(kubectl config view | grep https | grep 8443 | cut -f2- -d ':' | tr -d ' ')
	CERT=/home/justin/.minikube/profiles/minikube/client.crt
	KEY=/home/justin/.minikube/profiles/minikube/client.key
	curl -s $APISERVER --cert $CERT --key $KEY --insecure
	curl -s $APISERVER/version --cert $CERT --key $KEY --insecure
