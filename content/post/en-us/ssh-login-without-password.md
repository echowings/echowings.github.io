---
title: "Ssh Login Without Password"
date: 2018-11-29T00:22:57+08:00
lastmod: 2018-12-224T16:01:23+08:00
draft: false
tags: ["ssh"]
categories: ["linux"]
author: "Steve Dong"
reward: true
---


# Ssh Login Without Password

## SSH login linux/unix without password

## General your key

```
ssh-keygen-t rsa
```

## copy your rsa.pub to server which you want to login without password.

``` bash
ssh-copy-id -i ~/.ssh/id_rsa.pub user1@$server-hosts
```

## Test login

```bash
ssh user1@$server-hosts
```
