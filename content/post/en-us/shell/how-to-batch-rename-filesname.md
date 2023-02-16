---
title: "How to Batch Rename Filesname"
date: 2021-08-01T21:51:14+08:00
lastmode: 2021-08-01T21:51:14+08:00
draft: false
tags: [ "shell", "linux","chinese" ]
categories: ["shell"]

---

# How to batch rename filesname


## Rename a file
```bash
ls *.mp4| awk -F "TEXT_TO_DELETE" '{print "mv \""$0"\" \""$1$2"\""}'  | bash
```

## Rename a  folder

```bash
ls -d * | awk -F "TEXT_TO_DELETE"  '{print "mv \""$0"\" \""$1$2"\""}' | bash
```


## Batch rename file

  - bash file

```bash
#!/bin/bash
for dir in `ls . | tr " " "?" `
do 
  new_dir=(`echo $dir  | tr "?" " "`)
  if [ -d  """$new_dir""" ]
  then
    echo $new_dir
    cd """$new_dir""" 
    ls | awk -F "【瑞客论坛 www.ruike1.com】" '{print "mv \""$0"\" \""$1""$2"\""}' |bash  
    cd .. 
  fi
done

```
  - one command line script
 
```bash
for dir in `ls . | tr " " "?" `;do new_dir=(`echo $dir  | tr "?" " "`)&& (if [ -d  """$new_dir""" ];then;echo $new_dir&&cd """$new_dir""" &&ls | awk -F "【瑞客论坛 www.ruike1.com】" '{print "mv \""$0"\" \""$1""$2"\""}' |bash  && cd .. ;fi);done
```




