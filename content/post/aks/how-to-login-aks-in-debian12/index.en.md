---
title: "How to Login Aks in Debian 12"
date: 2024-03-06T14:45:21+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=090962a2
categories:
  - k8s
tags:
  - k8s
  - aks
  - debian
  - debian12
  - kubernetes
  - azcli

layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

## Install az-cli

```shell
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
zsh
```
## Install kubectl
```shell
#Install Kubectl
if [ ! -f /usr/local/bin/kubectl ]
then
	sudo curl -fsSL --progress-bar "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" -o /usr/local/bin/kubectl
sudo chmod +x /usr/local/bin/kubectl
fi

#Reload zsh
zsh 


#add alias into ~/.zshrd
if ! sudo grep -q "kubectl" ~/.zshrc; then
  cat << EOF |sudo tee -a  ~/.zshrc
alias k=kubectl

EOF
fi
zsh
cat ~/.zshrc | grep kubectl

```


## Get Contributer Privilege with PIM
Get the Contributer PIM firstly on azure portal
## Login aks

```shell
az login --use-device-code
az account set --subscription YOUR_SUBSCRIPTION
az aks get-credentials --resource-group RESOURCE-GROUP --name AKS-NAME --admin
kubectl config current-context
kubectl get po -A
```

## Reference
  - [Install the Azure CLI on Linux | Microsoft Learn](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt)
  - [Install and Set Up kubectl on Linux | Kubernetes](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
  - [Azure Kubernetes Error when running "az aks get-credentials" command - Stack Overflow](https://stackoverflow.com/questions/66033629/azure-kubernetes-error-when-running-az-aks-get-credentials-command)