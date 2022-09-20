---
title: "How to Compile Nginx With Source Code"
date: 2019-04-29T21:43:15+08:00
draft: false
tags: ["nginx","devops"]
categories: ["nginx"]
reward: true
---

# How to compile nginx with source code

## Download source code

 - Access [nginx.org](https://www.nginx.org) to download latest stable version of nginx. 
 
  ```shell
  wget  https://nginx.org/download/nginx-1.16.0.tar.gz
  tar -zxvf nginx-1.16.0.tar.gz
  ```
  
## Compile source code

  ```shell
  cd nginx-1.16.0
  apt update && apt install -y gcc build-essential
  ./configure --prefix=/home/echowings/nginx
  make
  make install
  
  ```

### Trouble shooting

  - PCRE missing error.

```shell
  ./configure: error: the HTTP rewrite module requires the PCRE library.

You can either disable the module by using —without–http_rewrite_module

option, or install the PCRE library into the system, or build the PCRE library

statically from the source with nginx by using —with–pcre=&lt;path&gt; option.
``` 

We need to install pcre library to support it.

 ```shell
 # Install pcre for rhel or centos
 yum install pcre-devel
 
 # Install pcre for debian or ubuntu
 apt install -y libpcre3 libpcre3-dev

 ```
 
   - gzip error.

  ```shell
  ./configure: error: the HTTP gzip module requires the zlib library.
You can either disable the module by using --without-http_gzip_module
option, or install the zlib library into the system, or build the zlib library
statically from the source with nginx by using --with-zlib=<path> option.
  ```
  
  We need to install gzip library.
  
  ```shell
  # For rhel or centos
  yum install zlib-devel 
  
  # For debian or ubuntu
  apt install zlib1g zlib1g-dev
  
  ```
   
 
 ## Reference
 
   - [How to install pcre and pcre-devel on ubuntu using apt-get?](https://www.techietown.info/2017/07/install-pcre-pcre-devel-ubuntu-using-apt-get/)
   - [./configure: error: the HTTP gzip module requires the zlib library.](http://www.voidcn.com/article/p-oyseyqkh-gs.html)
   - [Can't add nginx module - requires zlib](https://askubuntu.com/questions/980145/cant-add-nginx-module-requires-zlib/1081432#1081432)

