---
title: "How to Enable Http Api Service on VyOS"
date: 2022-04-26T00:17:15+08:00
lastmode: 2022-04-26T00:17:15+08:00
draft: false
tags: [ "vyos", "api" ]
categories: ["network"]
layout: "page"
slug: How to Enable Http Api Service on VyOS
reward: true
mathjax: true
image: https://picsum.photos/800/600.webp?random=71ff97d5
---

**How to enable API serivce on VyOS**

# How to enable API serivce on VyOS

After these days, I wrote some vyos shell script to auto detect network status and automatacally change some settings. In the way There will be new configuration to loading directly and hold the configuration mode for a longer time. This is not in a smart way.

Another better option, What if we can change settings of vyos with api?

## Enable api http service localy
### Enable apt service with socket mode 
```bash
set service https api socket 
```

### Test settings

```bash
curl --unix-socket /run/api.sock -X POST -Fkey=qwerty -Fdata='{"op": "showConfig", "path": []}' http://localhost/retrieve
```

## Enable api https service with let's encryption free ssl

```bash
#!/bin/vbash
source /opt/vyatta/etc/functions/script-template
configure

# Define variable
ID="my_id"
APIKEY="12345"
APIPORT="1234"
VHOST="myvps"
FULL_FQDN="xxx.xxx.xxx"
LISTEN_ADDRESS="IP ADDRESS"
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




