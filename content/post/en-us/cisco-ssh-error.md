---
title: "Cisco Ssh Error"
date: 2018-12-29T12:19:12+08:00
draft: false
tags: ["cisco", "ssh" ]
categories: ["ssh"]
author: Steve Dong
reward: true
---


# cisco ssh error

When you met error like this:

``` bash
ssh  cisco@10.32.3.1
Unable to negotiate with 10.32.3.1 port 22: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1
```

Solution:

``` bash
$ ssh -l <USERNAME> -oHostKeyAlgorithms=+ssh-dss -oKexAlgorithms=+diffie-hellman-group1-sha1 <HOST>
```
or

```bash
ssh -l <username> -o HostKeyAlgorithms=+ssh-dss -o KexAlgorithms=+diffie-hellman-group1-sha1 -c 3des-cbc <hostname>
```

## Reference
- [SSH failed on 3750 switch](https://community.cisco.com/t5/vpn-and-anyconnect/ssh-failed-on-3750-switch/td-p/3043810)
- [SSH Not Working After Update To 10.13.2 (HighSierra)](https://discussions.apple.com/thread/8196671)

