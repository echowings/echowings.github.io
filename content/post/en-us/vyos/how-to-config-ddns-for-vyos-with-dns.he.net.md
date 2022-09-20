---
title: "How to Config Ddns for Vyos With Dns.he.net"
date: 2020-08-05T16:26:46+08:00
lastmode: 2020-08-05T16:26:46+08:00
draft: false
tags: [ "vyos", "ddns","he.net" ]
categories: ["networking"]
reward: true
mathjax: true
---

# How to config DDNS for vyos with dns.he.net


## Register A Recored 
1. Register a domian `host.mydomain.com` on your [https://dns.he.net](https://dns.he.net)
2. Generate a key `64LGiLiuoiuoiuoi`



## Run command to setup the ddns service

```bash
configure

set service dns dynamic interface pppoe0 service dyndns host-name 'host.mydomain.com'
set service dns dynamic interface pppoe0 service dyndns login 'host.mydomain.com'
set service dns dynamic interface pppoe0 service dyndns password '64LGiLiuoiuoiuoi'
set service dns dynamic interface pppoe0 service dyndns server 'dyn.dns.he.net'
commit
save

```


