---
title: "How to Install Gitlab With Docker-Compose"
date: 2019-11-27T12:24:57+08:00
lastmode: 2019-11-27T12:24:57+08:00
draft: false
tags: [ "devops", "gitlab", "docker" ]
categories: ["devops"]
reward: true
mathjax: true
---

# How to Install gitlab with docer-compose



``` yaml
version: "3.5"
services:
  gitlab:
     image: gitlab/gitlab-ce:latest
     container_name: gitlab
     restart: always
     hostname: 'xa-gitlab'
     environment:
       GITLAB_OMNIBUS_CONFIG: |
         external_url 'http://xa-gitlab'
         gitlab_rails['backup_keep_time'] = 604800
         gitlab_rails['time_zone'] = 'Asia/Shanghai'
         gitlab_rails['gitlab_shell_ssh_port'] = 1022
         #gitlab_rails['ldap_enabled'] = true
         #gitlab_rails['ldap_servers'] = YAML.load <<-EOS
         #main:
         #  label: 'LDAP'
         #  host: '192.168.1.1'
         #  port: 389
         #  uid: 'uid'
         #  bind_dn: 'CN=admin,DC=lotbrick,DC=com'
         #  password: 'admin'
         #  user_filter: 'objectclass=person'
         #  base: 'ou=People,dc=xxxxx,dc=com'
         #  active_directory: false
         #  method: 'plain'
         #  attributes:
         #    username: ["cn"]
         #    email: ['email']
         #    name: 'displayName'
         #    first_name: 'givenName'
         #    last_name:  'sn'
         #EOS
     ports:
       - '80:80'
       - '443:443'
       - '1022:22'
     volumes:
       - ./gitlab/config:/etc/gitlab
       - ./gitlab/logs:/var/log/gitlab
       - ./gitlab/data:/var/opt/gitlab
       # - "/etc/localtime:/etc/localtime:ro"

```

## Reference
---
 - [Docker+GitLab+LDAP部署过程](https://www.jianshu.com/p/f7a9761eb9dd)
 - [https://github.com/JyBigBoss/docker-compose/blob/master/gitlab/docker-compose.yaml](https://github.com/JyBigBoss/docker-compose/blob/master/gitlab/docker-compose.yaml)