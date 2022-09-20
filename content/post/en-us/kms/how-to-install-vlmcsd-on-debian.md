---
title: "How to Install Vlmcsd on Debian"
date: 2021-03-02T12:19:36+08:00
lastmode: 2021-03-02T12:19:36+08:00
draft: false
tags: [ "vlmcsd", "kms","debian","vyos" ]
categories: ["vlmcsd"]
reward: true
mathjax: true
---

# How to Install VLMCSD on debian 

## VLMCSD 

  - [vlmcsd-1113-2020-03-28-Hotbird64](https://github.com/Wind4/vlmcsd/releases)

## Download and install


  ```bash
VLMCSD_VERSION=svn113
curl -fsSLO https://github.com/Wind4/vlmcsd/releases/download/$VLMCSD_VERSION/binaries.tar.gz
tar -zxvf binaries.tar.gz
sudo cp -af ./binaries/Linux/intel/glibc/vlmcsd-x64-glibc /usr/local/bin/
chown root:root /usr/local/bin/vlmcsd-x64-glibc
chmod +x /usr/local/bin/vlmcsd-64-glibc
sudo bash -c 'cat > /lib/systemd/system/vlmcsd.service' << EOF
[Unit]
Description=vlmcsd service

[Service]
ExecStart=/usr/local/bin/vlmcsd-x64-glibc -l /var/log/vlmcsd-64-glibc.log > /dev/null 2>&1
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
EOF


#Reload systemctl daemon
sudo systemctl daemon-reload

#Delete temp directory.
sudo rm -rf ./binaries
sudo rm -rf ./binaries.tar.gz

#Enable service of dcompass auto startup when system reboot
sudo systemctl enable vlmcsd

#Restart service of vlmcsd
sudo systemctl restart vlmcsd
#Check dcompass status of freedns.
sudo systemctl status vlmcsd
  
  ```

## bat file for client

```bat
echo "To active office"
cd "c:\Program Files\Microsoft Office\Office16"
cscript ospp.vbs /sethst:$HOTSNAMTORIP:$PORT
cscript ospp.vbs /inpkey:NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP
cscript ospp.vbs /act

echo "To active windows"
slmgr /skms $HOSTNAMEORIP:$PORT
slmgr /ato




@pause


```


## Reference

 1. [在debian中搭建kms服务器](https://invictuscj.win/articles/os/win-kms/)
  
  

