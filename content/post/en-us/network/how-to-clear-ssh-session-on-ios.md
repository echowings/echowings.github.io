---
title: "How to configure cisco catalyst switch"
date: 2021-05-11T11:29:28+08:00
lastmode: 2021-05-11T11:29:28+08:00
draft: false
tags: [ "ios", "cisco","catalyst", "network" ]
categories: ["network"]
reward: true
mathjax: true
---


# How to configure cisco catalyst switch

## Clean session

```bash
#Show current login users
show users

# terminal a user
clear line vty 0

```


## Set Session timeout

exec timeout command is sused to specify the timeout for exec sessions[telnet/ssh] whereas session timeout command specifies the idle timeout period for all the sessions. 

```bash
config t
line vty 0 15
exec-timeout 10
session-timeout 10

#Check configure
sh run | be line vty
```


## How to fix `%Error opening tftp://255.255.255.255/network-confg (Timed out)`

```bash
no service config
```


## Set ip address and default gateway

```bash
ip default-gateway 192.168.0.1

interface vlan 1
ip address 192.168.0.2 255.255.255.0

```

## Set hostname and domain-name

```bash
config t
hostname myswitch
ip domain-name mydomain.com
```

## Enable ssh login

### Generate the RSA keys
```bash
myswitch(config)# crypto key generate rsa
 The name for the keys will be: myswitch.thegeekstuff.com
 Choose the size of the key modulus in the range of 360 to 2048 for your
   General Purpose Keys. Choosing a key modulus greater than 512 may take
   a few minutes.

How many bits in the modulus [512]: 1024
 % Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
```


### Setup the line vty configurations

```bash
line vty 0 4
session-timeout 10
transport input ssh
login local
password 7
exit
```
### Set the console line
```bash
line console 0
logging synchronus
login local
```

### Create the username password

```bash
config t
username $USERNAME password $MYPASSWORD
enable secret $MYENABLEPASSWORD
```

### enable service password-encryption

```bash
service password-encryption
```

### Verify SSH access

```bash
show ip ssh
```



## Configure



### port-channel


```bash
interface Port-channel1
 description testing
 switchport trunk allowed vlan 4-12
 switchport trunk encapsulation dot1q
 switchport mode trunk
 
 
 
 # network port settings
 interface GigabitEthernet1/0/27
 description lacp-wifi-network
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 3 mode active
!
interface GigabitEthernet1/0/28
 description lacp-wifi-network
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 3 mode active
 
 
 
```

### Access mode

```bash
interface GigabitEthernet1/0/29
 switchport access vlan 60
 switchport mode access


```

### SNMP 

```bash
snmp-server community public RO
```

### NTP

```bash
ntp source vlan 20
ntp server 10.32.0.2 source vlan 20

```

### PBR

```bash

# Enable PBR on cisco 3750x
sdm prefer routing

#save configuration
write

#Reload switch to let it works.
reload

#check sdm status
show sdm prefer


# Define access list
ip access-list extended wifi
 permit ip 10.32.200.0 0.0.0.255 any
 deny   ip any any

#define reout-map and set routing policy
route-map wifi-2rd-gw permit 10
 match ip address wifi
 set ip next-hop 192.168.99.2
 
 
 #select a vlan to apply PBR
 interface Vlan30
 ip policy route-map wifi-2rd-gw
```

 


## Reference:
---
  1. [%Error opening tftp://255.255.255.255/network-confg (Timed out)](https://community.cisco.com/t5/other-network-architecture/error-opening-tftp-255-255-255-255-network-confg-timed-out/td-p/258876)
  2. [to increase SSH session timeout](https://community.cisco.com/t5/wireless/to-increase-ssh-session-timeout/td-p/3098020)
  3. [How to Enable SSH on Cisco Switch, Router and ASA](https://www.thegeekstuff.com/2013/08/enable-ssh-cisco/)







