---
title: "How to Update Network Group Base Domain Name on VyOS"
date: 2023-02-21T22:16:02+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=95b1deda
categories:
  - networking
tags:
  - vyos
  - shell
  - networking
  - automation
  - dns
  - domain
slug:
  - vyos
layout: 
  - post
license: CC BY-NC-ND
---


```bash
#!/bin/bash
set -e

# Domain list
domain_group="example-web"
domain_list=(auth0.example.com chat.example.com)
dns_server="114.114.114.114"
declare -a resolved_ips=()

TIMEOUT="10"

LOCAL_SERVER="localhost"
LOCAL_API_KEY="qwerty"

REMOTE_SERVER="API.YOUR.SERVER"
REMOTE_API_KEY="YOUR_API_KEY"
REMOTE_PORT="443"


# Pause function
function pause(){
 read -s -n 1 -p "Press any key to continue . . ."
 echo ""
}


echo "***************************************************"
echo "* Step 1. generate ${domain_group} domain to ip addresses *"
echo "***************************************************"

for domain in ${domain_list[*]};do
    ips=$(dig @${dns_server} $domain  +short | grep '^[.0-9]*$' | xargs)
    for ip in $ips;do
	echo $ip
	resolved_ips+=$ip
        resolved_ips+="/32 "
    done
done



echo "**************************************************************"
echo "* Step 2. Get ${domain_group} group in vyos with curl comman *"
echo "**************************************************************"
ips=$(curl -k  -s --unix-socket /run/api.sock \
    --connect-timeout ${TIMEOUT} \
    --location \
    --request POST "http://${LOCAL_SERVER}/retrieve"  \
    --form data='{"op": "showConfig", "path": ["firewall","group","network-group","'"${domain_group}"'","network"]}' \
    --form key="${LOCAL_API_KEY}"  | jq  '.data.network' | xargs)
local_ips=$(echo $ips | sed -e 's/\[ //g' -e 's/\ ]//g' -e 's/\,//g')
echo "local_ips: $local_ips"

ips=$(curl -k -s  --connect-timeout ${TIMEOUT} \
    --location --request POST "https://${REMOTE_SERVER}:${REMOTE_PORT}/retrieve" \
    --form data='{"op": "showConfig", "path": ["firewall","group","network-group","'"${domain_group}"'","network"]}'  --form key="${REMOTE_API_KEY}"  | jq '.data.network' | xargs)
remote_ips=$(echo $ips | sed -e 's/\[ //g' -e 's/\ ]//g' -e 's/\,//g')
echo "Network group named ${domain_group} on remote server: ${remote_ips}"


echo "*****************************************************"
echo "* Step 3. Update ${domain_group} with api operation *"
echo "*****************************************************"

# local_ips for vyos api returned chatgpt_group
# remote_ips for vyos api retruned remote vyos chatgpt group
# resolved_ips standard for dig return from dns server
echo "local_ips: $local_ips"
echo "resolved_ips: $resolved_ips"
echo "remote_ips: $remote_ips"
for ip_address in ${resolved_ips[*]};do
    if [[ "${local_ips[@]}" =~ ${ip_address} ]]
    then
        echo -e "included\n"
    else
        echo -e "Not included\n"
	echo "*********************************************************************"
	echo "* Add $ip_address into  network group ${domain_group} on local VyOS *"
	echo "*********************************************************************"
	echo "${LOCAL_SERVER}"
        curl -k -s --connect-timeout ${TIMEOUT} \
             --unix-socket /run/api.sock  \
             --location \
             -X POST "http://${LOCAL_SERVER}/configure" \
             -F data='{"op": "set", "path": ["firewall","group","network-group","'"${domain_group}"'","network", "'"${ip_address}"'"]}' \
             -F key=${LOCAL_API_KEY}
 	echo -e "/nsave configure./n"
        curl -k -s --connect-timeout ${TIMEOUT} \
             --unix-socket /run/api.sock  \
             --location \
             -X POST "http://${LOCAL_SERVER}/config-file" \
             -F data='{"op": "save"}' \
             -F key=${LOCAL_API_KEY}
        echo -e "/n"
    fi

    if [[ "${remote_ips[@]}" =~ ${ip_address} ]]
    then
        echo -e "included\n"
    else
	echo -e "not included\n"
	echo "**********************************************************************"
	echo "* Add $ip_address into  network group ${domain_group} on remote VyOS *"
	echo "**********************************************************************"
	curl -k -s  --connect-timeout ${TIMEOUT}  \
             -X POST -F key=${REMOTE_API_KEY} -F data='{"op": "set", "path": ["firewall","group","network-group","'"${domain_group}"'","network", "'"${ip_address}"'"]}'  \
              https://${REMOTE_SERVER}:${REMOTE_PORT}/configure

 	echo -e "/nsave configure./n"
	curl -k -s  --connect-timeout ${TIMEOUT}  \
             -X POST -F key=${REMOTE_API_KEY} -F data='{"op": "save"}'  \
              "https://${REMOTE_SERVER}:${REMOTE_PORT}/config-file"
        echo -e "/n"
    fi
done

```

