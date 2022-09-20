---
title: "Debian or Ubuntu Chinese Language Support"
date: 2018-12-24T00:39:51+08:00
lastmod: 2018-03-01T16:01:23+08:00
draft: false
tags: ["debian","ubuntu","linux","chinese"]
categories: ["linux"]
author: "Steve Dong"
reward: true
---

Ubuntu Chinese Language Support 

* 编辑/etc/environment如下:
  
  ```
  vi /etc/environment
  ```
添加如下:

  ```bash
  LC_ALL="en_US.UTF-8"
  LANG="zh_CN.UTF-8"
  UNZIP='-O CP936"
  ZIPINFO="-O CP936"
```

* 重新生成语言设置 

  ```bash
   sudo locale-gen  en_US en_US.UTF-8 zh_CN zh_CN.UTF-8
   sudo dpkg-reconfigure locales
```

## 中文支持

### vim中文支持

编辑~/.vimrc

``` bash
   set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
   set termencoding=utf-8
   set encoding=utf-8
```

### 中文支持

