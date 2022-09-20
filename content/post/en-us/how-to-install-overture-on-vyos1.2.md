---
title: "How to Install Overture on VYOS 1.2.1 release"
date: 2019-05-18T01:35:52+08:00
draft: false
tags: ["overture", "vyos"]
categories: ["dns"]
reward: true
---

# How to install overture on Vyos 1.2

I need to build a dns forwarding server to anti dns cache pollution.overture is a good choice for me. it writen in go with good performance.
Here we go!


# Download ovrture
you can check overture version from [here](https://github.com/shawn1m/overture)

 - Login vyos .

``` shell
sudo su 
overture_version=v1.6.1
sudo mkdir -p  /tmp/overture-linux-amd64
cd /tmp
sudo curl -fsSLO --compressed "https://github.com/shawn1m/overture/releases/download/$overture_version/overture-linux-amd64.zip"
sudo unzip /tmp/overture-linux-amd64.zip -d /tmp/overture-linux-amd64
cd /tmp/overture-linux-amd64

sudo curl -fsSLO https://raw.githubusercontent.com/17mon/china_ip_list/master/china_ip_list.txt

sudo curl -fsSLO https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt

curl https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt | base64 -d | sort -u | sed '/^$\|@@/d'| sed 's#!.\+##; s#|##g; s#@##g; s#http:\/\/##; s#https:\/\/##;' | sed '/\*/d; /apple\.com/d; /sina\.cn/d; /sina\.com\.cn/d; /baidu\.com/d; /qq\.com/d' | sed '/^[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+$/d' | grep '^[0-9a-zA-Z\.-]\+$' | grep '\.' | sed 's#^\.\+##' | sort -u > /tmp/temp_gfwlist.txt

curl https://raw.githubusercontent.com/hq450/fancyss/master/rules/gfwlist.conf | sed 's/ipset=\/\.//g; s/\/gfwlist//g; /^server/d' > /tmp/temp_koolshare.txt
cat /tmp/temp_gfwlist.txt /tmp/temp_koolshare.txt | sort -u > gfw_all_domain.txt

#Download china domain list from https://github.com/felixonmars/dnsmasq-china-list
curl -fsSLO --compressed https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/accelerated-domains.china.conf

#Downlaod apple china domain list
curl -fsSLO --compressed https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/apple.china.conf

#Download google china domain list
curl -fsSLO --compressed https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/google.china.conf

#Download cdn in china list
curl -fsSLO --compressed https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/cdn-testlist.txt


#
cat accelerated-domains.china.conf  cdn-testlist.txt google.china.conf apple.china.conf | sort -u >> china_domain.txt


# Strip useless characters in the configure file.
sed -i 's/server=\///g' china_domain.txt
sed -i 's/\/114.114.114.114//g' china_domain.txt




sudo mv config.json config.json.origin

sudo bash -c 'cat > /etc/overture/config.json' <<EOF
{
  "BindAddress": "127.0.1.1:53",
  "PrimaryDNS": [
    {
      "Name": "DNSPod",
      "Address": "119.29.29.29:53",
      "Protocol": "udp",
      "SOCKS5Address": "",
      "Timeout": 15,
      "EDNSClientSubnet": {
        "Policy": "disable",
        "ExternalIP": "",
        "NoCookie": true
      }
    }
  ],
  "AlternativeDNS": [
    {
      "Name": "OpenDNS",
      "Address": "208.67.222.222:53",
      "Protocol": "udp",
      "SOCKS5Address": "",
      "Timeout": 60,
      "EDNSClientSubnet": {
        "Policy": "disable",
        "ExternalIP": "",
        "NoCookie": true
      }
    }
  ],
  "OnlyPrimaryDNS": false,
  "IPv6UseAlternativeDNS": false,
  "WhenPrimaryDnsAnswerNoneUse": "AlternativeDNS",
  "IPNetworkFile": {
      "Primary": "/etc/overture/china_ip_list.txt"
   },
  "DomainFile": {
    "Primary": "/etc/overture/china_domain.txt",
    "Alternative": "/etc/overture/gfw_all_domain.txt",
    "PrimaryMatcher":  "suffix-tree",
    "AlternativeMatcher": "final"
  },
  "MinimumTTL": 60,
  "CacheSize" : 2000,
  "RejectQType": [255]
}
EOF

sudo cp -af ./overture-linux-amd64 /usr/local/bin/overture
sudo chown root:root /usr/local/bin/overture
sudo chmod +x /usr/local/bin/overture

sudo rsync -r --delete --exclude="overture-linux-amd64" --exclude="domain_sample" --exclude="ip_network_sample" --exclude="install.sh"  ./ /etc/overture

sudo bash -c 'cat > /lib/systemd/system/overture.service' << EOF
[Unit]
Description=overture service

[Service]
ExecStart=/usr/local/bin/overture -c /etc/overture/config.json -l /etc/overture/overture.log
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
EOF




sudo systemctl daemon-reload


sudo rm -rf /tmp/overture-linux-amd64
sudo rm -rf /tmp/overture-linux-amd64.zip
sudo systemctl enable overture
sudo systemctl restart overture


```


## Reference
---
  - [overture-on-openwrt](https://zhmin.github.io/2020/01/05/openwrt-v2ray/)


