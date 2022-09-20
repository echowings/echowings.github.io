---
title: "How to Install Freedns Go on Vyos"
date: 2021-01-11T01:13:07+08:00
lastmode: 2021-01-11T01:13:07+08:00
draft: false
tags: [ "vyos", "freedns","go" ]
categories: ["networking"]
reward: true
mathjax: true
---


```bash
#Switch to root mode
sudo su

#Define freedns version
freedns_version=v2020.9.17
#Create dcompass temp folder
sudo mkdir -p /tmp/freedns

# Swich to folder  just we created.
cd /tmp/freedns

#Download dcompass binary 
sudo curl -fsSLO "https://github.com/tuna/freedns-go/releases/download/$freedns_version/freedns-go-linux-amd64"



sudo cp -af /tmp/freedns/freedns-go-linux-amd64 /usr/local/bin/freedns
sudo chown root:root /usr/local/bin/freedns
sudo chmod +x /usr/local/bin/freedns

sudo bash -c 'cat > /lib/systemd/system/freedns.service' << EOF
[Unit]
Description=freedns service

[Service]
ExecStart=/usr/local/bin/freedns  -f 114.114.114.114:53 -c 8.8.8.8:53 -l 127.0.0.3:53
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
EOF

#Reload systemclt daemon
sudo systemctl daemon-reload

#Delete temp directory.
sudo rm -rf /tmp/freedns

#Enable service of dcompass auto startup when system reboot
sudo systemctl enable freedns

#Restart service of freedns
sudo systemctl restart freedns
#Check dcompass status of freedns.
sudo systemctl status freedns


```