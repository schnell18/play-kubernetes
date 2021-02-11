# Introduction

Nitty-gritty of minikube installation and usage.

## Installation

On Arch/Manjaro, you type:

	sudo pacman -S minikube

## Bootstrap a kubernetes cluster

You run:

	minikube start \
		--kubernetes-version=v1.19.2 \
		--image-mirror-country=cn\
		--image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
		--driver=virtualbox
