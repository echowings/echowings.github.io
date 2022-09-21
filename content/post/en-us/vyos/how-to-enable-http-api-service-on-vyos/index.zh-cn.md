---
title: "如何在vyos上启用api服务"
date: 2022-04-26T00:17:15+08:00
lastmode: 2022-04-26T00:17:15+08:00
draft: false
tags: [ "vyos", "api" ]
categories: ["networking"]
layout: "page"
slug: "page"
reward: true
mathjax: true
image: https://picsum.photos/800/600.webp?random={{ substr (md5 (.Date)) 4 8 }}
---


# 如何在VyOS上启用api服务

最近，我写了些shell脚本来自动探测网络状态，并通过脚本来更改vyos的设置，但是如果意外断电或者强制重启会导致VyOS的配置文件丢失，非常的麻烦和不稳定。
如何解决这个问题呢？为什么不实施VyOS的API呢？

## 在VyOS本机上启用api服务
### socket模式启用api服务
```bash
set service https api socket 
```

### curl测试

```bash
curl --unix-socket /run/api.sock -X POST -Fkey=qwerty -Fdata='{"op": "showConfig", "path": []}' http://localhost/retrieve
```

## 使用let‘s encryption的ssl启动VyOS API 服务

```bash
#!/bin/vbash
source /opt/vyatta/etc/functions/script-template
configure

# 定义变量
ID="my_id"
APIKEY="12345"
APIPORT="1234"
VHOST="myvps"
FULL_FQDN="xxx.xxx.xxx"
LISTEN_ADDRESS="xxx.xxx.xxx.xxx"
EMAIL="my@emal.com"

set service https api key id $ID key $APIKEY
set service https certificates certbot domain-name $FULL_FQDN
set service https certificates certbot email $EMAIL
set service https api strict
set service https virtual-host $VHOST listen-address $LISTEN_ADDRESS
set service https virtual-host $VHOST listen-port $APIPORT
set service https virtual-host $VHOST server-name $FULL_FQDN
commit
save
exit
```




