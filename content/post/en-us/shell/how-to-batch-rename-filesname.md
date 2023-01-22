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


## Rename a file
```bash
ls *.mp4| awk -F "TEXT_TO_DELETE" '{print "mv \""$0"\" \""$1$2"\""}'  | bash
```

## Rename a  folder

```bash
ls -d * | awk -F "TEXT_TO_DELETE"  '{print "mv \""$0"\" \""$1$2"\""}' | bash
```


## Reference