---
title: "Dockerfile for Overture"
date: 2018-12-24T00:55:14+08:00
lastmod: 2018-03-01T16:01:23+08:00
draft: false
tags: ["overture","docker"]
categories: ["docker"]
author: "Steve Dong"
reward: true
---


# How to Wirte docker file for overture
Hence we need to custom overture docker image as we desire, I write this dockerfile.

~~How to map VOLUME and copy file from docker images to directory of docker host.~~
~~When define VOLUME, build docker images will not copy file to the path, But we can copy the files when docker running.~~

~~if [ "`ls -A /config`" = "" ]; then cp /srv/config.json /config/ && cp /srv/*.txt /config/;fi~~

## Overture Dockerfile

``` dockfile
#FROM alpine:edge
FROM alpine:latest
MAINTAINER steve dong <dongjunbo@gmail.com?

# Environment variables
##########################

ENV VERSION v1.4

# Change this cron rule to what fits best for you.

ENV CRONTAB_TIME '0 0 * * *'

# Create Volume entry points
############################

VOLUME ["/config"]

# Set the work directory
#########################

WORKDIR /srv

# Fix permissions
#################

# Install required packages
###########################
RUN set -xe && \
	mkdir -p /config && \
	apk add --no-cache unzip curl && \
	curl -fsSLO --compressed "https://github.com/shawn1m/overture/releases/download/${VERSION}/overture-linux-amd64.zip" && \
	unzip -o "overture-linux-amd64.zip" -d /srv && \
	curl https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt | base64 -d | sort -u | sed '/^$\|@@/d'| sed 's#!.\+##; s#|##g; s#@##g; s#http:\/\/##; s#https:\/\/##;' | sed '/\*/d; /    apple\.com/d; /sina\.cn/d; /sina\.com\.cn/d; /baidu\.com/d; /qq\.com/d' | sed '/^[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+$/d' | grep '^[0-9a-zA-Z\.-]\+$' | grep '\.' | sed 's#^\.\+##' | sort -u > temp_gfwlist.txt && \
	curl https://raw.githubusercontent.com/hq450/fancyss/master/rules/gfwlist.conf | sed 's/ipset=\/\.//g; s/\/gfwlist//g; /^server/d' > temp_koolshare.txt  && \
	cat temp_gfwlist.txt temp_koolshare.txt | sort -u > gfw_all_domain.txt  && \
	curl -fsSLO --compressed  "https://raw.githubusercontent.com/17mon/china_ip_list/master/china_ip_list.txt" && \
	sed -i '11s/disable/manual/' /srv/config.json && \
	sed -i '24s/disable/manual/' /srv/config.json && \
	sed -i '12s/""/"222.90.74.60"/' /srv/config.json && \
	sed -i '25s/""/"202.60.225.114"/' /srv/config.json && \
	sed -i 's/".\/ip_network_primary_sample"/"\/config\/china_ip_list.txt"/g' /srv/config.json && \
	sed -i 's/".\/domain_alternative_sample"/"\/config\/gfw_all_domain.txt"/g' /srv/config.json && \
	rm -rf temp_koolshare.txt temp_gfwlist.txt && \
	rm -rf "overture-linux-amd64.zip" && \
	chmod 0644 /config/ && \
	cp /srv/config.json /config/ && \
	cp /srv/*.txt /config/ && \
	apk del unzip curl
	

# Expose required ports
#######################

EXPOSE 53/udp


#VOLUME [ "/config"]

# Change Shell
##############
SHELL ["/bin/bash", "-c"]
CMD ["if [ \"`ls -A /config`\" = \"\" ]; then cp /srv/config.json /config/ && cp /srv/*.txt /config/; fi"]
ENTRYPOINT [ "/srv/overture-linux-amd64", "-c", "/config/config.json" ]
#CMD /srv/overture-linux-amd64 -c /config/config.json
```

## Docker run Command

``` zsh
docker run  \
--restart=always \
-d \
-p 53:53/udp \
-v "/root/overture-docker-data:/config" \
--name xa-overture-01 \
overture

```