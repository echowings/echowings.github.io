---
title: "How to Disconnect and Reconnect Vyos Pppoe0 Interface With Crontab"
date: 2023-03-17T21:01:28+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=d4706f3e
categories:
  - networking
tags:
  - vyos
  - pppoe
  - crontab
  - vbash
slug:
  - vyos
layout: 
  - post

license: CC BY-NC-ND
---

## Create vbash script

### Create folder
```bash
mkdir /config/scripts/reboot_pppoe_connection
```

### Create script

```bash
mv /config/scripts/reboot_pppoe_connection/reboot_pppoe_connection.sh{,.bak}
ls /config/scripts/reboot_pppoe_connection/
tee -a  /config/scripts/reboot_pppoe_connection/reboot_pppoe_connection.sh << "EOF"
#!/bin/vbash
source /opt/vyatta/etc/functions/script-template
logfile=/var/log/reconnect_pppoe0.log

echo "==================================" >> ${logfile}  2>&1
date '+%Y-%m-%d %H:%M:%S' >> ${logfile} 2>&1
run show interfaces  | grep -A 2 ppoe0  | tee >> ${logfile} 2>&1
run disconnect interface pppoe0  | tee >> ${logfile} 2>&1
run connect interface pppoe0 | tee >> ${logfile} 2>&1
echo "sleep 3 second..." >> ${logfile} 2>&1
sleep 3
run  show interfaces  | grep -A 2 ppoe0 | tee >> ${logfile} 2>&1
date '+%Y-%m-%d %H:%M:%S' >>  ${logfile} 2>&1
echo "==================================" >> ${logfile} 2>&1
exit
EOF
```

### Chmod +x to vbash script

```bash
chmod +x /config/scripts/reboot_pppoe_connection/reboot_pppoe_connection.sh
```

### Create task with vyos command

```bash
config
set system task-scheduler task RUN_SHUTDOWN_PVE crontab-spec '00 3 * * *'
set system task-scheduler task RUN_SHUTDOWN_PVE executable path '/config/scripts/reboot_pppoe_connection/reboot_pppoe_connection.sh'
commit
save
```

