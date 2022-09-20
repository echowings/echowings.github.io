---
title: "How to Enable Ntfs Read and Write in Macos Catalina 10.15"
date: 2019-12-27T11:20:00+08:00
lastmode: 2019-12-27T11:20:00+08:00
draft: false
tags: [ "macos", "ntfs","catalina" ]
categories: ["macos"]
reward: true
mathjax: true
---

# How to enable NTFS Read and Write in Macos Catalina 10.15


For a long time. It was much trouble to enable ntfs write mode in macos. Today I  done it about enable ntfs wirte support with fuse and ntfs-3g. 



## Install FUSE for macos

You can download latest FUSE for macOS from here: [FUSE for macOS](https://github.com/osxfuse/osxfuse/releases)

Then you can install it by yourself.

## Install ntfs-3g

```bash

brew install ntfs-3g
```



## Replace ntfs-3g

Since from OS X 10.11 version ,macos enabled  system integrity protection we need to disable it to replace the `ntfs_mount`. 

So we gonna to disable System integrity Protection in Recovery mode ,then boot to normal mode to replace `ntfs_mount`, at last we need to boot to recovery mode again to enable system integrity protection.

### 1. Disable system integrity protection


  - Shutdown your macos
  - Press `Command + R`,the press power buttion to start macos and enter the recovery mode.
  - After enter `recovery mode`, then we can open Terminal.
  - Run command in Terminal `csrutil disable`.
  - Reboot macos.

### 2. Replace `mount_ntfs`

  - Open `Terminal`
  - Enter commands as show below

  ```bash
sudo mount -uw /
sudo mv /sbin/mount_ntfs /sbin/mount_ntfs.original
sudo ln -s /usr/local/sbin/mount_ntfs /sbin/mount_ntfs
  
  ```
  - Then shutdown your. macos.

  
### 3. Enable system integrity protection

 - Press `Command + R`,the press power buttion to start macos and enter the recovery mode.
 - After enter `recovery mode`, then we can open Terminal.
   Run command in Terminal `csrutil enble`.
 - Reboot macos.

 
 
### 4. Follow the system notification to enable fuse ntfs

After we boot macos normally, we need follow the system notification to enable permission for fuse ntfs.




##  Pro and Cons
**Pro:** We can write ntfs format disk for free.

**Cons:** We need to do it again when we upgrade macos to a new major release.


## Reference

  - [Enabling NTFS write in macOS 10.15 Catalina the Open source way](https://medium.com/macoclock/enabling-ntfs-write-in-macos-10-15-catalina-the-open-source-way-a5fd0d1cb32e)
  - [About System Integrity Protection on your Mac](https://support.apple.com/en-hk/HT204899)