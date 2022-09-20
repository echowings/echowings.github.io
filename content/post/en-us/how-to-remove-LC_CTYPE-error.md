---
title: "How to Remove LC_CTYPE Error"
date: 2018-11-29T23:48:32+08:00
lastmod: 2018-03-01T16:01:23+08:00
draft: false
tags: ["ubuntu","debian"]
categories: ["linux"]
author: "steve dong"
---



```
apt-get autoremove
export LC_ALL=en_US.UTF-8
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
```