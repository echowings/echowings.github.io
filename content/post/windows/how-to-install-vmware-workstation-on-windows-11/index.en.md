---
title: "How to Install Vmware Workstation on Windows 11"
date: 2024-02-20T15:57:44+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=11ca73ca
categories:
  - Blog
tags:
  - vmware
  - windows 11
slug:
  - test
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


```powershell

reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity" /v "Enabled" /t REG_DWORD /d 0 /f

bcdedit /set hypervisorlaunchtype off


dism /online /disable-feature /featurename:Microsoft-hyper-v-all

Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All

reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v "LsaCfgFlags" /t REG_DWORD /d 0 /f

reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard"  /v "EnableVirtualizationBasedSecurity" /t REG_DWORD /d 0 /f

reg add "HKLM\SYSTEM\Setup\LabConfig" /f

reg add "HKLM\SYSTEM\Setup\LabConfig" /v "BypassTPMCheck"  /t REG_DWORD /d 1 /f

reg add "HKLM\SYSTEM\Setup\LabConfig" /v "BypassSecureBootCheck"  /t REG_DWORD /d 1 /f

reg add "HKLM\SYSTEM\Setup\LabConfig" /v "BypassRAMCheck"  /t REG_DWORD /d 1 /f

```

## Reference
  - [How to Install Windows 11 on a VMware Virtual Machine](https://woshub.com/install-windows-11-vmware-vm/)
  - [How to disable Hyper-V in Windows 11](https://www.xda-developers.com/disable-hyper-v-windows-11/)
  - [Windows 11 Disable Hyper-V](https://www.makeuseof.com/windows-11-disable-hyper-v/)