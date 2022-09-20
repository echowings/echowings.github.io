---
title: "How to Install Snmp Service on Promxox VE"
date: 2019-04-27T20:29:36+08:00
draft: false
tags: ["pve","snmp","librenms"]
categories: ["virtualization"]
auther: "steve dong"
reward: false
---

# How to install snmp service on proxmox ve


How to setup snmp service to let us to use librenms to moniting our proxmox ve healthy status is very important for us.


## Install snmpd on proxmox ve

``` shell
apt update
apt install -y snmpd

```

## Configure snmpd.conf

  -  backup original configuration file.

```shell

mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.backup

```

  - Create a new config file.

``` shell
cat > /etc/snmp/snmpd.conf << EOF
# this create a  SNMPv1/SNMPv2c community named ""
# and restricts access to LAN adresses 192.168.0.0/22 (last two 0's are ranges)
rocommunity OkayWorld 192.168.0.0/22
# setup info
syslocation  "rack 1, room 3, serverrroom"
syscontact  "xxx@xxx.xxx"
# open up
agentAddress  udp:161
# run as
agentuser  root
# dont log connection from UDP:
dontLogTCPWrappersConnects yes
# fix for disks larger then 2TB
realStorageUnits 0
EOF

```

## Firewall allow lan can access

```shell
iptables -A INPUT -s 192.168.0.0/22 -p udp --dport 161 -j ACCEPT

```

## Restart snmpd service to apply settings

``` bash

systemctl restart snmpd

```

## Testing

``` bash
snmpwalk -c OkayWorld -v1 $servername$ SNMPv2-MIB::sysDescr.0

```

## QA
###1. error on subcontainer 'ia_addr' insert (-1)
After systemd migration starting from Debian 9 Stretch, you need to change `/lib/systemd/system/snmpd.service` file, not `/etc/default/snmpd` file:

``` bash
sed -i "s|-Lsd|-LS6d|" /lib/systemd/system/snmpd.service
systemctl daemon-reload
systemctl restart  snmpd
```

## Reference

 - [how to install SNMP service on Proxmox](https://www.svennd.be/how-to-install-snmp-service-on-proxmox/)
 - [How can I disable snmpd ia_addr insert messages?](https://linux-tips.com/t/how-can-i-disable-snmpd-ia-addr-insert-messages/281/2)
 - [[Solved] snmpd gives “error on subcontainer” ‘ia_addr’ insert (-1) warnings
](http://www.leonli.co.uk/blog/487/solved-snmpd-gives-error-on-subcontainer-ia_addr-insert-1-warnings)



