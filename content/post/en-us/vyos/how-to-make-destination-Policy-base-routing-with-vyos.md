---
title: "How to make Destination Base Routing With Vyos"
date: 2020-09-03T16:36:31+08:00
lastmode: 2020-09-03T16:36:31+08:00
draft: false
tags: [ "vyos", "PBR","network_shunt" ]
categories: ["networking"]
reward: true
mathjax: true
---

# How to make Destination Base Routing with Vyos



I have 2 network interfaces. one for user access inboard network,another for user to access network aboard. How to make it work with vyos?

## Make a network group

```bash
#Define a netwrok-group
set firewall group network-group china-ip-ranges

#Define lan network group
set firewall group network-group lan-net-group network '192.168.11.0/24'
```

## set policy

```bash
set policy route network-shunt rule 20 destination group network-group 'china-ip-ranges'
set policy route network-shunt rule 20 set table '20'
set policy route network-shunt rule 20 source group network-group 'lan-net-group'
set policy route network-shunt rule 30 set table '30'
set policy route network-shunt rule 30 source group network-group 'lan-net-group'

```

## Download and set the network-group with ipset

 - Download and make a list for network-group loading when booting.

 
```bash
#create the update shell script
cat  > /config/scripts/generate_china_ip_address.sh <<EOF
#!/bin/sh
wget -c http://ftp.apnic.net/stats/apnic/delegated-apnic-latest -O - | cat | awk -F '|' '/CN/&&/ipv4/ {print $4 "/" 32-log($5)/log(2)}' | cat > /config/chinaiprange.txt
EOF

# make the script an run
chmod +x /config/scripts/generate_china_ip_address.sh 

#Run the script
/config/scripts/generate_china_ip_address.sh 

```

- Create boot loading script

```bash
cat >> /config/scripts/vyos-postconfig-bootup.script << EOF
for l in `cat /config/chinaiprange.txt`; do sudo ipset add china-ip-ranges $l;done
EOF

```


## Set protocol

```bash
set protocols static interface-route 0.0.0.0/0 next-hop-interface pppoe0
set protocols static interface-route 172.16.131.0/24 next-hop-interface wg01
set protocols static table 20 interface-route 0.0.0.0/0 next-hop-interface pppoe0
set protocols static table 30 route 0.0.0.0/0 next-hop 172.16.131.1
```

## Apply PBR Policy on lan interface


```bash
set interfaces bridge br0 policy route network-shunt
```



