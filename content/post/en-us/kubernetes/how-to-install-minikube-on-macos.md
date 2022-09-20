---
title: "How to Install Minikube on Macos"
date: 2021-01-20T15:27:49+08:00
lastmode: 2021-01-20T15:27:49+08:00
draft: true
tags: [ "minikube", "kubernetes","macos" ]
categories: ["kubernetes"]
reward: true
mathjax: true
---

# How to Install minikube on macos


## Install script

## 

```bash
brew install minikube
```

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-hyperkit \
&&chmod +x docker-machine-driver-hyperkit \
&&sudo mv docker-machine-driver-hyperkit /usr/local/bin/ \
&&sudo chown root:wheel /usr/local/bin/docker-machine-driver-hyperkit \
&&sudo chmod u+s /usr/local/bin/docker-machine-driver-hyperkit
```

