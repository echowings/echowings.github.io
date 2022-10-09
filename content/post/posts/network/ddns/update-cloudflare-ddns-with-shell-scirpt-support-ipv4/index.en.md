---
title: "Shell Script to Update Cloudflare Ddns"
date: 2022-10-09T09:50:14+08:00
draft: false
tags: [ "ddns", "cloudflare" ]
categories: ["network"]
layout: "page"
slug: "Shell Script to Update Cloudflare Ddns"
image: https://picsum.photos/800/600.webp?random=7257319e

---

# Shell Script to Update Cloudflare Ddns


## Script
```bash
#!/bin/bash


##############  User Configuration  ###############
# Define programme path
# location_path="/config/scripts/ddns"
location_path=$(dirname -- "$0")

# You Registered CloudFlare  Mail Account
auth_email=""

# CloudFlare Global API Key
auth_key=""
# Root Doman Name
zone_name=""

# IP Type，Enter 'ipv4' or 'ipv6' ,at here we got paramter from our input
type=$1

# DDNS Domain Name
record_name=$2

# WAN INTERFACE
INTERFACE=""

#############  Script  ############
# The before WAN IP 
ip_file="$location_path/$1_$2_ip.txt"

# Domain id information
# Domain id information
id_file="$location_path/$1_$2_cloudflare.ids"

# Log location
log_file="$location_path/cloudflare.log"

################  Don't change here  ###############
##################  Function  ####################
record_type=""
ip=""
zone_identifier=""
record_identifier=""
update=""

# Log function
log() {
    if [ "$1" ]; then
        echo -e "[$(date)] - $1" >> $log_file
    fi
}
#To get local IP address
get_ip() {
    if [ $type == "ipv4" ]; then
	    record_type="A"
	    ip=$(ip addr | grep ${INTERFACE}  -A2 | grep inet | grep -v inet6  | awk '{print $2}')
    elif [ $type == "ipv6" ]; then
	    record_type="AAAA"
	    ip=$(ip addr | grep ${INTERFACE}  -A2 | grep inet6  | awk '{print $2}' | cut -d '/' -f1)
    else
	    echo "Type wrong"
	    log "Type wrong"
        exit 0
    fi
}

# Check IP changed or not, if there's no change, then terminal programme
check_ip_change() {
    if [ -f $ip_file ]; then
        old_ip=$(cat $ip_file)
        if [ "$ip" == "$old_ip" ]; then
            echo "IP has not changed."
            log "IP has not changed."
            exit 0
        fi
    fi
}
#Get zone_id subdomain ID
get_id() {
    if [ -f $id_file ] && [ $(wc -l $id_file | cut -d " " -f 1) == 2 ]; then
        zone_identifier=$(head -1 $id_file)
        record_identifier=$(tail -1 $id_file)
    else
        zone_identifier=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=$zone_name" -H "X-Auth-Email: $auth_email" -H "X-Auth-Key: $auth_key" -H "Content-Type: application/json" | grep -Po '(?<="id":")[^"]*' | head -1 )
        rec_response_json=`curl -X GET "https://api.cloudflare.com/client/v4/zones/$zone_identifier/dns_records?name=$record_name" -H "X-Auth-Email: $auth_email" -H "X-Auth-Key: $auth_key" -H "Content-Type: application/json"`
        record_identifier=`echo $rec_response_json | grep -Po '(?<="id":")[^"]*'`
        echo "$zone_identifier" > $id_file
        echo "$record_identifier" >> $id_file
    fi
}
#Update DNS record
update_dns() {
    update=$(curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$zone_identifier/dns_records/$record_identifier" -H "X-Auth-Email: $auth_email" -H "X-Auth-Key: $auth_key" -H "Content-Type: application/json" --data "{\"id\":\"$zone_identifier\",\"type\":\"$record_type\",\"name\":\"$record_name\",\"content\":\"$ip\"}")
}
###################  Main  ###################
log "Script start."
#Got IP Address
get_ip

# Check got the right ip address
if [ "$ip" == "" ]; then
    echo "Can not get IP address.Please check your network connection."
    log "Can not get IP address.Please check your network connection."
    exit 0
fi

# Check IP changed or not
check_ip_change


# Get zone_id and sumdomain record ID
get_id

# Check get ID successfully or not
if [ "$zone_identifier" == "" ]; then
    echo "Can not get zone_id."
    log "Can not get zone_id."
    exit 0
elif [ "$record_identifier" == "" ]; then
    echo "Can not get record_id."
    log "Can not get record_id."
    exit 0
fi

# Update DNS Record
update_dns

# Check DNS record updated or not
if [[ $update == *"\"success\":false"* ]]; then
    log "API UPDATE FAILED. DUMPING RESULTS:\n$update"
    echo -e "API UPDATE FAILED. DUMPING RESULTS:\n$update"
    exit 0
else
    echo "$ip" > $ip_file
    log "$record_name IP changed to: $ip"
    echo "$record_name IP changed to: $ip"
fi
```


## How to run the script


```bash
#update A record
sudo bash /config/scripts/ddns/cloudflare-ddns.sh ipv4 <MY_DOMAIN_NAME>

#update AAAA record
sudo bash /config/scripts/ddns/cloudflare-ddns.sh ipv4 <MY_DOMAIN_NAME>
```
