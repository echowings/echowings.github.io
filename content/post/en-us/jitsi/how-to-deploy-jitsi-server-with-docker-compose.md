---
title: "How to Deploy Jitsi Server With Docker Compose"
date: 2020-05-21T22:49:44+08:00
lastmode: 2020-05-21T22:49:44+08:00
draft: flase
tags: [ "docker", "docker-compose","jitsi" ]
categories: ["linux"]
reward: true
mathjax: true
---

# How to deploy Jisti Server with docker-compose

## Install jisti Server

 1. git clone latest version 
	
```bash
git clone https://github.com/jitsi/docker-jitsi-meet && cd docker-jitsi-meet
	
cp env.example .env
```
	
 2. Generate security password

```bash
./gen-passwords.sh
#mkdir -p ~/.jitsi-meet-cfg/{web/letsencrypt,transcripts,prosody,jicofo,jvb}
mkdir -p ~/.jitsi-meet-cfg/{web/letsencrypt,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}
``` 

## Modify `.env` 

1. copy `env.example` to `.env`.

```bash
cp env.example .env
```
2. Modify key and value as followed.

```bash
CONFIG=~/.jitsi-meet-cfg

# Exposed HTTP port
HTTP_PORT=80

# Exposed HTTPS port
HTTPS_PORT=443

# System time zone
TZ=Asia/Shanghai

# Public URL for the web service
PUBLIC_URL=https://meet.mydomain.com

#
# Let's Encrypt configuration
#

# Enable Let's Encrypt certificate generation
ENABLE_LETSENCRYPT=1

# Domain for which to generate the certificate
LETSENCRYPT_DOMAIN=meet.mydomain.com

# E-Mail for receiving important account notifications (mandatory)
LETSENCRYPT_EMAIL=myname@mydomain.com

# Enable authentication
ENABLE_AUTH=1

# Enable guest access
ENABLE_GUESTS=1

# Select authentication type: internal, jwt or ldap
AUTH_TYPE=internal

# Redirect HTTP traffic to HTTPS
# Necessary for Let's Encrypt, relies on standard HTTPS port (443)
ENABLE_HTTP_REDIRECT=1

``` 

## Add Users for  internal auth

```bash
docker exec -it docker-jitsi-meet_prosody_1 bash
# create a user `user1`
prosodyctl --config /config/prosody.cfg.lua register user1 meet.jitsi 
```
## Run it

```bash
docker-compose up -d

```
## Backup

```bash
#for configure folder
~/.jitsi-meet-cfg
# for letencryption folder
~/.jitsi-meet-cfg/web/letsencrypt
# for configure
./.env

```
## Firewall Setup

```bash
apt install ufw
ufw status
ufw default deny
ufw allow 22
ufw allow 443
ufw allow 80
ufw allow 10000/udp
ufw allow 4443
ufw enable
ufw status
```
 
 

## Reference
* [Quick start](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker)
* [用docker安裝視訊會議 jitsi-meet](https://kafeiou.pw/2020/04/04/2354/)


