---
title: "How to Build Ipsec Site to Site Vpn With Vyos and Pfsense"
date: 2019-12-19T14:36:09+08:00
lastmode: 2019-12-19T14:36:09+08:00
draft: false
tags: [ "vyos", "pfsense","ipsec", "site-to-site" ]
categories: ["networking"]
reward: true
mathjax: true
---

# How to build ipse site-to-site VPN with vyos and pfsense


## Vyos VS pfsense

For build all kinds of network functions like nat,firewall, site-to-site vpn , dial in vpn with pfsense.It works well. But it has some Crons:

  1. Hard to upgrade version, it manybe crashed
  2. Didn't support API.
  3. Heavy loading and low performance.

  
After try to replace pfsense to build  all functions with vyos. It has a lot of pros:

  1. Easy upgrade and rollback
  2. Support http and python api
  3. Support ansible deploy
  4. Command line operation,just like juniper and cisco.
 

 
Today I will show you how to build ipsec site-to-site vpn with vyos and pfsense
 
## Prerequest

 
| | SITE A | SITE B|
|---|---|---|
|WAN ADDRESS | `1.1.1.1` | `2.2.2.2` |
|WAN Interface | `eth0` | `eth0` |
|LAN ADDRESS | `192.168.1.1/24` | `192.168.2.1/24` |
|LAN NETWORK |`192.168.1.0/24` | `192.168.2.0/24`  |
|LAN Interface | `eth1` | `eth1` |
|OS | `Vyos` | `pfsense` |
|Pre-Shared Key |  `*#JCenaoewaoi0298J299*8&^(%9))_&&` | `*#JCenaoewaoi0298J299*8&^(%9))_&&` |



##  Configure vyos 

Configure `Site A` 
 
```bash
configure
set vpn ipsec ipsec-interfaces interface eth0
set vpn ipsec ike-group IKE-Default dead-peer-detection action 'hold'
set vpn ipsec ike-group IKE-Default dead-peer-detection interval '30'
set vpn ipsec ike-group IKE-Default dead-peer-detection timeout '120'
set vpn ipsec ike-group IKE-Default ikev2-reauth 'no'
set vpn ipsec ike-group IKE-Default key-exchange 'ikev2'
set vpn ipsec ike-group IKE-Default lifetime '10800'
set vpn ipsec ike-group IKE-Default mobike 'disable'
set vpn ipsec ike-group IKE-Default proposal 1 dh-group '14'
set vpn ipsec ike-group IKE-Default proposal 1 encryption 'aes256gcm128'
set vpn ipsec ike-group IKE-Default proposal 1 hash 'sha256'

set vpn ipsec esp-group ESP-Default compression 'disable'
set vpn ipsec esp-group ESP-Default lifetime '3600'
set vpn ipsec esp-group ESP-Default mode 'tunnel'
set vpn ipsec esp-group ESP-Default pfs 'dh-group14'
set vpn ipsec esp-group ESP-Default proposal 1 encryption 'aes256gcm128'
set vpn ipsec esp-group ESP-Default proposal 1 hash 'sha256'

set vpn ipsec site-to-site peer 2.2.2.2 authentication id '1.1.1.1'
set vpn ipsec site-to-site peer 2.2.2.2 authentication mode 'pre-shared-secret'
set vpn ipsec site-to-site peer 2.2.2.2 authentication pre-shared-secret '*#JCenaoewaoi0298J299*8&^(%9))_&&'
set vpn ipsec site-to-site peer 2.2.2.2 authentication remote-id '2.2.2.2'
set vpn ipsec site-to-site peer 2.2.2.2 connection-type 'initiate'
set vpn ipsec site-to-site peer 2.2.2.2 ike-group 'IKE-Default'
set vpn ipsec site-to-site peer 2.2.2.2 ikev2-reauth 'inherit'
set vpn ipsec site-to-site peer 2.2.2.2 local-address '1.1.1.1'
set vpn ipsec site-to-site peer 2.2.2.2 tunnel 0 esp-group 'ESP-Default'
set vpn ipsec site-to-site peer 2.2.2.2 tunnel 0 local prefix '192.168.1.0/24'
set vpn ipsec site-to-site peer 2.2.2.2 tunnel 0 remote prefix '192.168.2.0/24'
```
 
 
## Configure pfsense

Configure `Site B`

1. After login pfsense click `VPN` |  `IPsec`;
2. Then click 'Add P1` button.
3. configuraton as show below


### Add P1 


General Information
---
|Key | Value |
|---|---|
|`Disabled` | uncheck |
|`Key Exchange version` | IKEv2|
|Internet Protocol | IPv4|
|Interfae |WAN|
|Remote Gateway | 1.1.1.1|
|Description| SITE-A-VPN|
 
  
Phase 1 Proposal (Authentication)
---

| Key | Value |
|---|---|
|Authentication Method | Mutual PSK |
| My identifier | My IP address |
| Peer identifier | Peer IP address |
| Pre-Shared Key | `*#JCenaoewaoi0298J299*8&^(%9))_&&` |
  
 
Phase 1 Proposal(Encryption Algorithm)
---
  
| Key | Algorithm| Key length | Hash | DH Group |
|---|---|---|---|---|
|Encryption Algorithm | AES256-GCM | 128 bits | SHA256 | 14(2048bit) |
  
|Key | Value|
|---|---|
|Lifetime(Seconds) | 28800 |
  
   
Advanced Options
---
| KEY  | VALUE |
|---|---|
|Disable rekey | uncheck|
|Margintime(Seconds | black| 
|Disable Reauth | uncheck |
|Responder Only | Uncheck |
| MOBIKE | Disable |
| Split connections | uncheck |
| Dead Peer Dection | 'Check on' Enable DPD |
|Delay | 120 |
|Max failures | 30 |
  
  
### Add P2

General Information
---

|Key | Value |
|---|---|
|Disable | unchecked |
|Mode | Tunnel IPv4 |
|Local network | Network  192.168.2.0/24|
|NAT/BINAT translation | None |
|Remote Network | Network 192.168.1.0/24|
| Description | Site A|

Phase 2 Proposal(SA/Key Exchange)
---

|Key | Value|
|---|---|
|Protocol | ESP |
|Encryption Algorithms | AES256-GCM 128 bit |
|Hash Algorithms | SHA256 |
|PFS key group | 14(2048 bit) |
|Lifttime | 3600 |

Advanced Configuration
---



|Key | Value |
|---|---|
|Automatically ping host | 192.168.1.1 |


### Change firewall to allow ipsec vpn connection



| Protocol | Source | Port | Destination | Port |
|---|---|---|---|---|
|Ipv4 UDP |1.1.1.1 |* |WAN address | 500(ISAKMP) |
|IPv4 UDP | 1.1.1.1 | * |WAN address | 4500(IPsec NAT-T) |
|Ipv4 ESP | 1.1.1.1 | * | WAN address | * |


---The End---
  
  
  


 
 
