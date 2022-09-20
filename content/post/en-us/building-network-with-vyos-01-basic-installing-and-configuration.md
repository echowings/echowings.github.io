---
title: "Building Network With Vyos 01---Basic Installing and Configuration"
date: 2018-10-26T22:44:02+08:00
tags: ["vyos"]
categories: [ "networking" ]
author: "steve dong"
draft: false
reward: true
---

# Building Network With Vyos.-.01.Basic Installing and Configuration

Vyos is a debian base router. it is fork form vyatta core.It is very stable and fast router OS. It hasn’t web UI, but only command line configration just like JUNOS.

At now, vyos version will be vyos version 1.2 released. I will wirte the turtural with vyos 1.2RC4.

## Install vyos

* Download vyos form the office website:

  * Download Link: [vyos-1.2.0-rc-4-amd64](https://downloads.vyos.io/testing/1.2.0-rc4/vyos-1.2.0-rc4-amd64.iso)
  
## Write iso image to usb disk

 * Linux and Mac: 

``` bash
ddi if=vyos-1.2.0-rc-4-amd64.iso of=/dev/$yourusbdisk bs=1M
```

 * Windows:

 1. download Rufus: [**rufus**](https://rufus.ie/en_IE)
 2. select ``dd`` mode to write disk.

## Install vyos with wizard.
 1. press `F11` or set COMS to boot from usb.

 2. Enter Default username and password to login livecd.

	```bash
vyos login: vyos
Password: vyos
```

## Write image to local driver:


```bash
install image
```

  * Reboot System to boot from local disk.

  ```bash
reboot now
```


## Basic Command

vyos command line include 2 mode: Operation mode and configure mode.

  * Operation Mode 
shell command start with $. 
These command usually show sysytem. information,reboot/shutdown system.

  * Configure Mode 
 shell command start with #. 
 These command usually change configuration of vyos.


### Operation Mode


``` shell
# Enter configure mode
configure


#Exit to operation Mode
exit
#Check vyos version
show version
#Check vyos running time
show system uptime
#Check system time and timezone
show date
#Reboot system
reboot now
#Shutdown System
poweroff now
#upgrade system image version
add system image $URL_or_path


#Reset configuration
# Enter config mode
configure
#loading default configure
load /opt/vyatta/etc/config.boot.default  
# commit loading default configure
commit

# save to default boot configure file.
save

#Exit configure mode
exit

#Reboot system now to finish loading default configuration.
reboot now



#bonding

```

<conitune...>