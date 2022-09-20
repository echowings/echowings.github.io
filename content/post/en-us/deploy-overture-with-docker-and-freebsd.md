---
title: "Deploy Overture With Docker and Freebsd"
date: 2018-12-18T00:44:13+08:00
lastmod: 2018-03-01T16:01:23+08:00
draft: false
tags: ["docker","freebsd","overture","dns"]
categories: ["docker","networking"]
author: "Steve Dong"
reward: true
---


# Run overture on Docker and FreeBSD 12

``` dockerfile
ENV foo /overture
WORKDIR ${foo}
RUN apt-get update && apt-get install -y ca-certificates

RUN  printf  "deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse\n deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse\n deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse\n deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse\n deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse\n deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse\n deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse\n deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse"  > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y wget curl unzip apt-utils vim
RUN cd /overture
RUN wget https://github.com/shawn1m/overture/releases/download/v1.5-rc1/overture-linux-amd64.zip
RUN unzip overture-linux-amd64.zip
RUN curl https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt | base64 -d | sort -u | sed '/^$\|@@/d'| sed 's#!.\+##; s#|##g; s#@##g; s#http:\/\/##; s#https:\/\/##;' | sed '/\*/d; /apple\.com/d; /sina\.cn/d; /sina\.com\.cn/d; /baidu\.com/d; /qq\.com/d' | sed '/^[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+$/d' | grep '^[0-9a-zA-Z\.-]\+$' | grep '\.' | sed 's#^\.\+##' | sort -u > /tmp/temp_gfwlist.txt
RUN curl https://raw.githubusercontent.com/hq450/fancyss/master/rules/gfwlist.conf | sed 's/ipset=\/\.//g; s/\/gfwlist//g; /^server/d' > /tmp/temp_koolshare.txt
RUN cat /tmp/temp_gfwlist.txt /tmp/temp_koolshare.txt | sort -u > gfw_all_domain.txt
RUN wget https://raw.githubusercontent.com/17mon/china_ip_list/master/china_ip_list.txt
RUN printf '{ \n   "BindAddress": ":53",\n   "PrimaryDNS": [\n    {\n      "Name": "DNSPod",\n      "Address": "114.114.114.114:53",\n      "Protocol": "udp",\n      "SOCKS5Address": "",\n       "Timeout": 6,\n       "EDNSClientSubnet": {\n         "Policy": "auto",\n         "ExternalIP": "222.90.74.60",\n         "NoCookie": true\n       }\n     }\n   ],\n   "AlternativeDNS": [\n     {\n       "Name": "OpenDNS",\n       "Address": "208.67.222.222:443",\n       "Protocol": "tcp",\n       "SOCKS5Address": "",\n       "Timeout": 6,\n       "EDNSClientSubnet": {\n         "Policy": "manual",\n         "ExternalIP": "202.60.225.114",\n         "NoCookie": true\n       }\n     }\n   ],\n   "OnlyPrimaryDNS": false,\n   "IPv6UseAlternativeDNS": false,\n   "IPNetworkFile": {\n     "Primary": "./china_ip_list.txt",\n     "Alternative": "./ip_network_alternative_sample"\n   },\n   "DomainFile": {\n     "Primary": "./domain_primary_sample",\n     "Alternative": "./gfw_all_domain.txt"\n   },\n   "HostsFile": "./hosts_sample",\n   "MinimumTTL": 0,\n   "CacheSize" : 0,\n  "RejectQtype": [255]\n }' > config.json

CMD ["./overture-linux-amd64"]
#ENTRYPOINT ["./overture-linux-amd64"]
```


## FreeBSD rc.d file

`cat /usr/local/etc/rc.d/overture`

```rc.d
#!/bin/sh
# $FreeBSD: head/net/overture/files/overture.in 335066 2018-12-18 11:33:26z SteveDong $

# PROVIDE: overture
# REQUIRE: LOGIN cleanvar
# KEYWORD: shutdown

# Add the following lines to /etc/rc.conf to enable overture
# overture_enable (bool): set to "NO" by default
# Set to "YES" to enable overture.
# overture_config(path): overture config file.
# Default to "/usr/local/etc/overture/config.json"

. /etc/rc.subr

name="overture"
rcvar=overture_enable


load_rc_config $name

: ${overture_enable:="NO"}
: ${overture_user:="root"}
: ${overture_group:="nobody"}

#: ${error_log_file="/var/log/overture_error"}
#: ${overture_args:="-p ${cpu_core} -c ${overture_config} -l ${logfile}"}
: ${overture_config="/usr/local/etc/overture/config.json"}



command="/usr/sbin/daemon"
procname="/usr/local/bin/overture"
pidfile="/var/run/overture.pid"
logfile="/var/log/overture"
error_log_file="/var/log/overture_error"
requried_files="${overture_config}"

# find how many cores on your computer

cpu_core=$(cat /var/run/dmesg.boot | grep CPU | grep FreeBSD | awk {'print $5'})

overture_args="-p ${cpu_core} -c ${overture_config} -l ${logfile} 2>&1 > $error_log_file"

command_args="-p ${pidfile}  /usr/bin/env ${procname} ${overture_args}"

start_precmd=overture_startprecmd

overture_startprecmd()
{
	if [ ! -e ${pidfile} ]; then
		install -o ${overture_user} -g ${overture_group} /dev/null ${pidfile};
	fi
	if [ ! -e ${logfile} ]; then
		install -o ${overture_user} -g ${overture_group} /dev/null ${logfile};
	fi
	if [ ! -e ${error_log_file} ]; then
		install -o ${overture_user} -g ${overture_group} /dev/null ${error_log_file};
	fi
}

load_rc_config $name
run_rc_command "$1"
```