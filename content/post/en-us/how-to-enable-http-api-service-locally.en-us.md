---
title: "How to Enable Http Api Service Locally"
date: 2022-04-26T00:17:15+08:00
lastmode: 2022-04-26T00:17:15+08:00
draft: false
tags: [ "vyos", "api" ]
categories: ["networking"]
reward: true
mathjax: true
image: https://picsum.photos/800/600.webp?random={{ substr (md5 (.Date)) 4 8 }}
slug:
  - myslug
---

**How to enable API serivce on VyOS**

## How to enable API serivce on VyOS

After these days, I wrote some vyos shell script to auto detect network status and automatacally change some settings. In the way There will be new configuration to loading directly and hold the configuration mode for a longer time. This is not in a smart way.

Another better option, What if we can change settings of vyos with api?

## enable api http service localy
### Enable apt service with socket mode 
```bash
set service https api socket 
```

### Test settings

```bash
curl --unix-socket /run/api.sock -X POST -Fkey=qwerty -Fdata='{"op": "showConfig", "path": []}' http://localhost/retrieve
```




