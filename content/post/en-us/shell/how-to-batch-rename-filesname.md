---
title: "How to Batch Rename Filesname"
date: 2021-08-01T21:51:14+08:00
lastmode: 2021-08-01T21:51:14+08:00
draft: false
tags: [ "shell", "linux","chinese" ]
categories: ["shell"]
reward: true
mathjax: true
---

#How to batch rename filesname

```bash
ls *.mp4| awk -F" 【中文 www.zhongwen.zw 】" '{print "mv \""$0"\" \""$1$2"\""}'  | bash:
```