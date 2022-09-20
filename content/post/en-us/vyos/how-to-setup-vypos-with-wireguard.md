---
title: "How to Setup Vypos With Wireguard"
date: 2021-01-03T16:30:43+08:00
lastmode: 2021-01-03T16:30:43+08:00
draft: true
tags: [ "vyos", "wireguard","networking","linux" ]
categories: ["networking"]
reward: true
mathjax: true
---


## Setup VyOS basic

### 1. set banner

```bash
set system login banner post-login '\nWelcome to VyOS!\n'
set system login banner pre-login '\n\nUNAUTHORIZED USE OF THIS SYSTEM\nIS PROHIBITED!\n\n'
```
### 2. Set Interfaces

```bash
#Set WAN
set interfaces ethernet eth0 pppoe 0 user-id 029021722680
set interfaces ethernet eth0 pppoe 0 password a1b2c3
set interfaces ethernet eth0 pppoe 0 name-server none

# Set LAN Bridge
set interfaces bridge br0 address 192.168.50.1/24
set interfaces bridge br0 description LAN-bridge
set interfaces ethernet eth1 bridge-group bridge br0
set interfaces ethernet eth2 bridge-group bridge br0
set interfaces ethernet eth3 bridge-group bridge br0
set interfaces ethernet eth4 bridge-group bridge br0
set interfaces ethernet eth5 bridge-group bridge br0

```

### 3. Set NAT

```bash
set nat source rule 100 outbound-interface pppoe0
set nat source rule 100 translation address masquerade
```

### 4. Set Default routing


```bash
set protocols static interface-route 0.0.0.0/0 next-hop-interface pppoe0
```

### 5. Set DNS Server

```bash
set service dns forwarding allow-from 192.168.50.0/24
set service dns forwarding allow-from 127.0.0.1/32
set service dns forwarding listen-address 192.168.50.1
set service dns forwarding listen-address 127.0.0.1
set service dns forwarding name-serve 114.114.114.114

```

### 6. Set System DNS Server

```bash
set system name-server 114.114.114.114
set system name-server 127.0.0.1
```

### 7. Set DHCP Server

```bash
set service dhcp-server shared-network-name lan-network subnet 192.168.50.0/24 default-router 192.168.50.1
set service dhcp-server shared-network-name lan-network subnet 192.168.50.0/24 dns-server 192.168.100.1
set service dhcp-server shared-network-name lan-network subnet 192.168.50.0/24 range 0 start 192.168.50.50
set service dhcp-server shared-network-name lan-network subnet 192.168.50.0/24 range 0 stop 192.168.50.254
```

### 8. Set DDNS

```bash
set service dns dynamic interface pppoe0 service dyndns host-name nobuo.stevedong.com
set service dns dynamic interface pppoe0 service dyndns login nobuo.stevedong.com
set service dns dynamic interface pppoe0 service dyndns password kA4qsQxO4ffCm7Be
set service dns dynamic interface pppoe0 service dyndns server dyndns.he.net
```

### 9. Set system user

```bash
delete system login user vyos
set system login user gcadmin level admin
set system login user gcadmin authentication plaintext-password *#92702689#SA

```

### 10. Set hostname

```bash
set system host-name nobuo-gw-gateway

```

### 11. Set Timezone

```bash
set system time-zone Asia/Chongqing 
```
### 12. Set ssh service

```bash
set service ssh port 9528
```

## Set Wireguard VPN

```bash


```


