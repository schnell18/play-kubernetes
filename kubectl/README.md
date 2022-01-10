# Introduction

Nitty-gritty of kubectl installation and usage.

## Installation

To install kubectl on Linux, download the latest stable kubectl binary, make it
executable and move it to the PATH:

    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" && chmod +x kubectl && sudo mv kubectl /usr/local/bin/

Where https://storage.googleapis.com/kubernetes-release/release/stable.txt aims
to display the latest Kubernetes stable release version. To download and setup
a specific version of kubectl (such as v1.19.2), issue the following command:

    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.19.2/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/

## Setup bash autocompletion

A typical helpful post-installation configuration is to enable shell
autocompletion for kubectl. It can be achieved by running the following four
commands:

    sudo apt install -y bash-completion
    source /usr/share/bash-completion/bash-completion
    source <(kubectl completion bash)
    echo 'source <(kubectl completion bash)' >>~/.bashrc
