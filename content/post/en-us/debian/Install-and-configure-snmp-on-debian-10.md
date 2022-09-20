---
title: "Install and Configure Snmp on Debian 10"
date: 2021-01-22T13:31:14+08:00
lastmode: 2021-01-22T13:31:14+08:00
draft: false
tags: [ "debian", "debian10","snmp" ]
categories: ["linux"]
reward: true
mathjax: true
---

# Install and Configure snmp on debian 10


## INSTALL SNMP	
```bash

apt install -y snmpd snmp libsnmp-dev

```

## CONFIGURE SNMP ON DEBIAN 10

 -  backup original configure file. <br>

 
```bash
cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.orig

```

 - Edit snmpd.conf

 
 ```bash
 
 agentAddress udp:127.0.0.1:161,udp:192.168.1.1:161
 
 ```