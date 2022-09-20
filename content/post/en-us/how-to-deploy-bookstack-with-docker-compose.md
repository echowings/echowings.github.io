---
title: "How to Deploy Bookstack With Docker Compose"
date: 2019-07-30T16:22:52+08:00
lastmode: 2019-07-30T16:22:52+08:00
draft: False
tags: [ "bookstack", "document", "docker" ]
categories: ["Linux"]
reward: true
mathjax: true
---

# How to Deploy Bookstack with Docker Compose

Bookstack is a good opensource solution for enterprie document management with light footprinting. It support auth like ldap,active directory,google,azure,aws,slack etc. and it can export document as pdf file.


## Preparation

  - [Install docker  and docker-compose on debian 10](https://blog.stevedong.com/post/how-to-configure-debian-10-as-a-docker-server/)
  - Information for bookstack installation.

|Type | Value|
|---|---|
|**Hostname**| mybookstack|
|**Domainame** | mydomain.local |
|**AD Auth User**| stevedong|
|**Domain Controller IP** | \<domain contoller IP>|
|**Time Zone** | Asia/Shanghai |



### How to fix export chinese pdf document with error

 1.  Install chinese fonts on docker server host

 ```bash
apt install fonts-wqy-microhei fonts-wqy-zenhei xfonts-wqy
```

 2. customized dockerfile with option:

 ```yaml
 environments:
    # To let bookstack export pdf with chinese language supported.
    - WKHTMLTOPDF=/usr/bin/wkhtmltopdf
    # To let bookstack export pdf with chinese language supported.
    - LANG=C.UTF-8
 volumes:
	# To let bookstack export pdf with chinese language supported.
    - /usr/share/fonts/truetype:/usr/share/fonts/truetype
 ```

### Enable Active Directory Auth


```bash

- AUTH_METHOD=ldap
# LDAP Settings
- LDAP_SERVER=ldap://<Domain_controller_IP>:389
- LDAP_BASE_DN=dc=mydomain,dc=local
- LDAP_DN=stevedong@mydomain.local
- LDAP_PASS=<password of AD USER>
#use double $$ since we are using docker
- LDAP_USER_FILTER=(&(sAMAccountName=$${user}))
- LDAP_VERSION=3
- LDAP_EMAIL_ATTRIBUT=mail
- LDAP_DISPLAY_NAME_ATTRIBUTE=cn

```








## 

save follow content to `Docker-compose.yml`

```yaml
bookstack:
    image: linuxserver/bookstack
    container_name: bookstack
    environment:
      - TZ=Asia/Shanghai
      - PUID=1000
      - PGID=1000
      - DB_HOST=bookstack_db
      - DB_USER=bookstack
      - DB_PASS=<changepassword>
      - DB_DATABASE=bookstackapp
      - APP_ENV=production
      - APP_DEBUG=false
      #- APP_LANG=zh_CN
      - APP_URL=http://mybookstack.mydomain.local
      # General auth
      # Change to local auth with replace `ldap` with `standard`
      - AUTH_METHOD=ldap
      # LDAP Settings
      - LDAP_SERVER=ldap://<Domain_controller_IP>:389
      - LDAP_BASE_DN=dc=mydomain,dc=local
      #- LDAP_DN=cn=stevedong,dc=mydomain,dc=local
      - LDAP_DN=stevedong@mydomain.local
      - LDAP_PASS=<password of AD USER>
      #use double $$ since we are using docker
      - LDAP_USER_FILTER=(&(sAMAccountName=$${user}))
      - LDAP_VERSION=3
      - LDAP_EMAIL_ATTRIBUT=mail
      #- LDAP_EMAIL_ATTRIBUT=false
      - LDAP_DISPLAY_NAME_ATTRIBUTE=cn
      #AD Group Sync
      - LDAP_USER_TO_GROUPS=true
      - LDAP_GROUP_ATTRIBUTE="memberof"
      - LDAP_REMOVE_FROM_GROUPS=false
      - LDAP_AUTO_CONFIRM_EMAIL=true
      - LDAP_TLS_INSECURE=true
      # To let bookstack export pdf with chinese language supported.
      - WKHTMLTOPDF=/usr/bin/wkhtmltopdf
      # To let bookstack export pdf with chinese language supported.
      - LANG=C.UTF-8
    volumes:
      - ./bookstack_config:/config
      #mapped docker server fonts to docker
      - /usr/share/fonts/truetype:/usr/share/fonts/truetype
    ports:
      - 80:80
    restart: unless-stopped
    depends_on:
      - bookstack_db
  bookstack_db:
    image: linuxserver/mariadb
    container_name: bookstack_db
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=<mysql password>
      - TZ=Asia/Shanghai
      - MYSQL_DATABASE=bookstackapp
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=<mysql password>
    volumes:
      - ./db_config:/config
    restart: unless-stopped
```

### Run Docker-compose
  - change value to fit your environment
  - Run bookstack with command `docker-compose up -d`
  - Test till  you can login bookstack with an active directory user account. 

### Change AD user role from viewer to Admin

  - stop docker-compose `docker-compose down`
  - Modify `Docker-compose.yml` to change  `- AUTH_METHOD=ldap` to `- AUTH_METHOD=standard`
  - Start docker-compsoe `docker-compsoe up -d`
  - Login bookstack with username `admin@admin.com` and password `password`.Then change your AD user account role from guest to admin.
  - Then stop docker with command `docker-compsoe down`
  - Modify `Docker-compose.yml` to change `-AUTH_METHOD=standard` back to `-AUTH_METHOD=ldap`
  - Start bookstack dockers with command `docker-composed up -d`.
  - Then you can login bookstack with your AD account as an administrator.






## Reference

  - [solidnerd/docker-bookstack](https://github.com/solidnerd/docker-bookstack)
  - [linuxserver/docker-bookstack](https://github.com/linuxserver/docker-bookstack)
  - [Install and Configure BookStack Using Docker and LDAP](https://blog.rylander.io/2017/06/09/install-and-configure-bookstack-using-docker-and-ldap/)
  - [successage/bookstack](https://hub.docker.com/r/successage/bookstack)