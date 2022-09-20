---
title: "How to Install Dcompass on Vyos"
date: 2020-12-22T02:57:46+08:00
lastmode: 2020-12-22T02:57:46+08:00
draft: false
tags: [ "vyos", "debian","dcompass" ]
categories: ["linux"]
reward: true
mathjax: true
---


## Dcompass github address

1. [Dcompass Address](https://github.com/LEXUGE/dcompass)
2. [Dcompass Release bianry](https://github.com/LEXUGE/dcompass/releases)

```bash
#Switch to root mode
sudo su

#Define dcompass version
dcompass_version=build-20210803_1007

#Create dcompass temp folder
sudo mkdir -p /tmp/dcompass

# Swich to folder  just we created.
cd /tmp/dcompass

#Download dcompass binary 
sudo curl -fsSLO "https://github.com/LEXUGE/dcompass/releases/download/$dcompass_version/dcompass-x86_64-unknown-linux-musl-full"

#Create dcompass configure folder
sudo mkdir -p /etc/dcompass

#Create dcompass configure yaml file
sudo bash -c 'cat > /etc/dcompass/config.yaml' <<EOF
---
verbosity: "info"
#address: 0.0.0.0:2053
address: 127.0.1.2:53
table:
  start:
    if:
      qtype:
        - AAAA
    then:
      # A list of actions is allowed here
      - blackhole
      # The next tag to go
      - end
    else:
      - dispatch
  dispatch:
    if: any
    then:
      - query: domestic
      - check_secure
  check_secure:
    if:
      geoip:
        codes:
          - CN
    else:
      - query: aboard
      - end

upstreams:
  114DNS:
    udp:
      addr: 114.114.114.114:53

  Ali:
    udp:
      addr: 223.6.6.6:53

  domestic:
    hybrid:
      - 114DNS
      - Ali
  googledns1:
    udp:
      addr: 8.8.8.8:53

  googledns2:
    udp:
      addr: 8.8.4.4:53
  aboard:
    hybrid:
      - googledns1
      - googledns2
EOF

sudo cp -af /tmp/dcompass/dcompass-x86_64-unknown-linux-musl-full /usr/local/bin/dcompass
sudo chown root:root /usr/local/bin/dcompass
sudo chmod +x /usr/local/bin/dcompass

sudo bash -c 'cat > /lib/systemd/system/dcompass.service' << EOF
[Unit]
Description=dcompass service

[Service]
ExecStart=/usr/local/bin/dcompass -c /etc/dcompass/config.yaml 
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
EOF

#Reload systemclt daemon
sudo systemctl daemon-reload

#Delete temp directory.
sudo rm -rf /tmp/dcompass

#Enable service of dcompass auto startup when system reboot
sudo systemctl enable dcompass

#Restart service of dcompass
sudo systemctl restart dcompass
#Check dcompass status of dcompass.
sudo systemctl status dcompass




```
			