# Introduction

Nitty-gritty of minikube installation and usage.

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

