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
        --kubernetes-version=v1.19.2 \
        --image-mirror-country=cn\
        --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
        --driver=virtualbox

Substitute the kubernetes version as you see fit.

## Launch dashboard

You run:

    minikube dashboard

and minikube will launch a browser and bring you to the dashboard webui.

