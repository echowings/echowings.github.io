---
title: "How to Convert Cert to Ssl"
date: 2020-01-21T16:49:32+08:00
lastmode: 2020-01-21T16:49:32+08:00
draft: false
tags: [ "nginx", "ssl", "openssl" ]
categories: ["linux"]
reward: true
mathjax: true
---


# How to convert godaddy certification files from `.cert` to `.pem`

 
 Recently days, I met an issue to renew ssl certification file for a website. The former developer setup certification file to `.pem`. It take a long time to make it work. Here we go.
 

 
## Convert `.crt` to `.pem`

``` bash

# convert .key file to key.pem
openssl rsa -in mywebsite.com.key -text > key.pem

# convert .cert to cert.pem
cat mywebsite.com.crt > cert.pem

# add cert chain to the cert.pem
cat mywebsite.com.ca-bundle >> cert.pem

# clone pem
cp cert.pem staging.pem

#
openssl dhparam -dsaparam -out /etc/nginx/ssl/dhparam.pem 4096
tar -cxvf tar -jcvf mywebsite.com-v3.tar.bz2 *.pem

```

## Upload `mywebsite.com-v3.tar.bz2` to server and renew ssl certification

```bash

tar -jxvf mywebsite.com-v3.tar.bz2
mv all the pem file to /home/mywebsitepath

# Testing nginx configuration
docker exec -it nginx nginx -t

# Restart nginx instance
docker restart nginx

```

