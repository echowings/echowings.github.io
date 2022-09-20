---
title: "How to Setup Vyos as a Firewall"
date: 2019-08-08T13:58:35+08:00
lastmode: 2019-08-08T13:58:35+08:00
draft: false
tags: [ "vyos", "firewall","network" ]
categories: ["networking"]
reward: true
mathjax: true
---

# How to Setup Vyos as a Firewall


## Setup interface

| Interface|IP Address |Description|
|---|---|---|
|eth0|pppoe  | WAN|
|eth1|192.168.68.1 / 24 | DMZ|
|eth2 | 10.0.0.1 / 24 | LAN|


```bash
set interface ethernet eth0 pppoe 0 user-id '<pppoe accout>'
set interface ethernet eth0 pppoe 0 password '<pppoe password>'
set interface ethernet eth0 pppoe 0 name-server 'none'
set interface ethernet eth0 description 'WAN'

set interface ethernet eth1 address 192.168.68.1/24
set interface ethernet eth1 description DMZ

set interface ethernet eth2 address 10.0.0.1/24
set interface ethernet eth2 description 'LAN'
```


## DHCP Server
 
```bash
set shared-network-name lan.local subnet 192.168.68.0/24 default-router 192.168.68.1
set shared-network-name lan.local subnet 192.168.68.0/24 dns-server 192.168.68.1
set shared-network-name lan.local subnet 192.168.68.0/24 domain-name lan.local
set shared-network-name lan.local subnet 192.168.68.0/24 range 0 start 192.168.50
set shared-network-name lan.local subnet 192.168.68.0/24 range 0 stop 192.168.68.254'

```

## DNS forwarding


```bash
set cache-size '0'
set listen-address 192.168.68.1
set name-server <public dns server ip ,like 8.8.8.8>

```

## Set loging banner

You are able to set post-login or pre-login messages with the following lines:

```bash
set system login banner pre-login "UNAUTHORIZED USE OF THIS SYSTEM IS PROHIBITED\n"
set system login banner post-login "Welcome to VyOS"
```

## NAT

### SNAT
```bash
set source rule 10 outbound-interface pppoe0
set source rule 10 source address 192.168.68.0/24
set source rule 10 translation address masquerade

```

### DNAT

```bash
set nat destination rule 10 destination address '10.0.0.5'
set nat destination rule 10 destination port '3389'
set nat destination rule 10 inbound-interface 'eth2'
set nat destination rule 10 protocol 'tcp_udp'
set nat destination rule 10 translation address '192.168.68.50'
set nat destination rule 10 translation port '3389'
```

## Default routing

```bash
set static interface-route 0.0.0.0/0 next-hop-interface pppoe0
```

## Setting of firewall

### Zone base firewall
```bash

# Chang firewall to statue firewall
set firewall state-policy established action 'accept'
set firewall state-policy related action 'accept'
set firewall state-policy invalid action 'drop'

# Set Zone for interfaces
set zone-policy zone dmz interface 'eth1'
set zone-policy zone private interface 'eth2'
set zone-policy zone local local-zone
set zone-policy zone public interface 'pppoe0'

# define firewall policy
set firewall name dmz-local default-action 'drop'
set firewall name dmz-local rule 10 action 'accept'
set firewall name dmz-local rule 10 destination address '192.168.68.1'
set firewall name dmz-local rule 10 destination port '53'
set firewall name dmz-local rule 10 protocol 'udp'

set firewall name dmz-local rule 11 action 'accept'
set firewall name dmz-local rule 11 icmp type-name 'echo-request'
set firewall name dmz-local rule 11 protocol 'icmp'

set firewall name dmz-private default-action 'drop'
set firewall name dmz-private rule 10 action 'accept'
set firewall name dmz-private rule 10 destination address '192.168.68.1'
set firewall name dmz-private rule 10 destination port '53'
set firewall name dmz-private rule 10 protocol 'udp'

set firewall name dmz-public default-action 'accept'


set firewall name local-dmz default-action 'accept'

set firewall name local-private default-action 'accept'

set firewall name local-public default-action 'accept'

set firewall name private-dmz default-action 'accept'

set firewall name private-local default-action 'accept'

# Allow ping
set firewall name private-local rule 1 action 'accept'
set firewall name private-local rule 1 icmp type-name 'echo-request'
set firewall name private-local rule 1 protocol 'icmp'

set firewall name private-public default-action 'accept'

set firewall name public-dmz default-action 'drop'

set firewall name public-local default-action 'drop'

set firewall name public-private default-action 'drop'





set zone-policy zone dmz default-action 'drop'
set zone-policy zone private default-action 'drop'
set zone-policy zone public default-action 'drop'

set zone-policy zone dmz from local firewall name 'local-dmz'
set zone-policy zone dmz from private firewall name 'private-dmz'
set zone-policy zone dmz from public firewall name 'public-dmz'

set zone-policy zone local from dmz firewall name 'dmz-local'
set zone-policy zone local from private firewall name 'private-local'
set zone-policy zone local from public firewall name 'public-local'


set zone-policy zone private from dmz firewall name 'dmz-private'
set zone-policy zone private from local firewall name 'local-private'
set zone-policy zone private from public firewall name 'public-private'

set zone-policy zone public from dmz firewall name 'dmz-public'
set zone-policy zone public from local firewall name 'local-public'
set zone-policy zone public from private firewall name 'private-public'



set firewall all-ping 'enable'
set firewall broadcast-ping 'disable'
set firewall config-trap 'disable'
set firewall receive-redirects 'disable'
set firewall send-redirects 'enable'
set firewall source-validation 'disable'
set firewall syn-cookies 'enable'
set firewall twa-hazards-protection 'disable'
set firewall ip-src-route 'disable'
set firewall ipv6-receive-redirects 'disable'
set firewall ipv6-src-route 'disable'
set firewall log-martians 'enable'


commit
save


```




