---
title: "How to Uninstall Windows Update Which Install Failed"
date: 2020-12-29T11:50:20+08:00
lastmode: 2020-12-29T11:50:20+08:00
draft: false
tags: [ "windows", "blue screen","repair boot" ]
categories: ["windows"]
reward: true
mathjax: true
---

# How to uninstlal windows update which install failed

---


## Uninstall failed windows update

 1. Reboot your pc or laptop and select `troubleshoot`
![](https://blog.workinghardinit.work/wp-content/uploads/2017/10/image_thumb-1.png)

 2. Select to open the command prompt and stay away from any other auto repair options.
![](https://blog.workinghardinit.work/wp-content/uploads/2017/10/image-2.png)
 3. Loading the software rigistry.
 
 ```bash
 #Load the softeware registry hive as follows.
 reg load hklm/temp c:\windows\system32\config\software
 
 #Delete the sessionPending registry key, if it exists by running:
 reg delete "HKLM\temp\Microsoft\Windows\CurrentVersion\Component Based Servicing\SessionPending" /v Exclusive
 
 # Unload the software registry hive:
 reg unload HKLM\temp
 ```
 
 
## Remove failed updates with `DISM` command

 1. List failed updated.

 The yellow one are the ones of interest and you can see the first one never even got an install time.

 ```bash
#List updates installed that caused issue.
dism /image:c:\ /get-packages
```

 ![](https://blog.workinghardinit.work/wp-content/uploads/2017/10/image-3.png)
 2. Remove error udpates with `DISM`.

 ```bash
 dism /image:c:\ /remove-package /packagename:myproblematicpackagetoremove /scratchdir:c:\temp
 ```
 ![](https://blog.workinghardinit.work/wp-content/uploads/2017/10/image-4.png)
 3. After finished uninstall error updates,we can reboot your pc.

 ![](https://blog.workinghardinit.work/wp-content/uploads/2017/10/image-5.png)
 



## Reference
---

1. [Quick Fix Publish : VM won’t boot after October 2017 Updates for Windows Server 2016 and Windows 10 (KB4041691)](https://blog.workinghardinit.work/2017/10/11/quick-fix-publish-vm-wont-boot-after-october-2017-updates-for-windows-server-2016-and-windows-10-kb4041691/)

