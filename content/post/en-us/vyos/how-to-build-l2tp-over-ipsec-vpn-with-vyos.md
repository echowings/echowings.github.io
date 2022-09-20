---
title: "How to Build L2tp Over Ipsec VPN With Vyos"
date: 2020-08-05T10:57:22+08:00
lastmode: 2020-08-05T10:57:22+08:00
draft: false
tags: [ "vyos ", "vpn","pppoe", "adsl" ]
categories: ["networking"]
reward: true
mathjax: true
---

# How to build L2tp over ipsec vpn with vyos

Today, I will show you how to build a l2tp over ipsec vpn with adsl wan address.

### Setup vpn on vyos 
---

### Generate a preshare key 
---


```bash
$ generate wireguard preshared-key
uND0O6RYI1O833NL2+qwSrW4htMll1hJAUM2nPQaa8k=
```


### With authentication local
---

1. configuration VPN 

|Key | Value|
|---|---|
|WAN interface | pppoe0|
|l2tp vpn network| 172.16.100.0/24|
|PRESHARE KEY | uND0O6RYI1O833NL2+qwSrW4htMll1hJAUM2nPQaa8k=|  
|DNS server 01 |192.168.0.1|
|DNS Server 02 | 1921.68.0.2 |



```bash
configure
set vpn ipsec auto-update '30'
set vpn ipsec ipsec-interfaces interface 'pppoe0'
set vpn ipsec nat-networks allowed-network 0.0.0.0/0
set vpn ipsec nat-traversal 'enable'

set vpn l2tp remote-access authentication mode 'local'
set vpn l2tp remote-access authentication require 'mschap-v2'
set vpn l2tp remote-access client-ip-pool start '172.16.100.1'
set vpn l2tp remote-access client-ip-pool stop '172.16.100.254'
set vpn l2tp remote-access dns-servers server-1 '192.168.0.1'
set vpn l2tp remote-access ipsec-settings authentication mode 'pre-shared-secret'
set vpn l2tp remote-access ipsec-settings authentication pre-shared-secret 'uND0O6RYI1O833NL2+qwSrW4htMll1hJAUM2nPQaa8k='
set vpn l2tp remote-access ipsec-settings ike-lifetime '3600'
set vpn l2tp remote-access ipsec-settings lifetime '3600'
set vpn l2tp remote-access outside-address '0.0.0.0'

```



###  Auth  users  to access vpn
---

1. Generate password

```bash
dd if=/dev/urandom bs=1 count=15 | base64
```
2. create user named `zhangsan` with password `J+Yi8o5ICLjKtezv0+i9`

```bash
configure
set vpn l2tp remote-acces	authentication local-users username zhangshan password J+Yi8o5ICLjKtezv0+i9
commit
save

```

### Start the service 

---

```bash
sudo systemctl enable xl2tpd
sudo systemctl restart xl2tpd

```


## vpn setup script for windows 10


```bat

@echo off&PUSHD %~DP0 &TITLE ikev2 VPN InstallUninstall programme
mode con cols=160 lines=50
set TempFile_Name=%SystemRoot%\System32\BatTestUACin_SysRt%Random%.batemp
( echo "BAT Test UAC in Temp" >%TempFile_Name% ) 1>nul 2>nul
if exist %TempFile_Name% (
del %TempFile_Name% 1>nul 2>nul&&goto :install
) else (
echo;Please run the script as administrator,Press any key to quit... &&goto :end

)

echo;VPN List:
echo; Welcome! This programme will help your install MY-VPN, or uninstall it. 

:install
cls

echo;1:MY-VPN(L2tp/ipsec)
echo;Please select Install or Uninstall VPN(ikev2)(Enter 'i' for install, 'u' for uninstall, 'q' for quit):
echo;i:Install MY-VPN(L2TP/IPSEC VPN)
echo;u:Uninstall MY-VPN(L2TP/IPSEC VPN)
echo;q:Quit
set/p choose1=Please enter your selection and press enter key to confirm:
echo %choose1%|findstr /i "[iuq]">nul&&goto :%choose1%
goto :install

:i
cls
echo;Install 1:MY-VPN(L2TP/IPSEC VPN)
echo;===========================
echo;***Step 1:Delete VPN with the same name ***
(powershell -Command "& {Remove-VpnConnection -Name "MY-VPN" -Force -PassThru;}") 1>nul 2>nul
echo;Successfully!
echo;
echo;***Step 2: Install new VPN***
(powershell -Command "& {Enable-PSRemoting -Force;}") 1>nul 2>nul
(powershell -Command "& {Add-VpnConnection -Name "MY-VPN" -ServerAddress "myvpn.mydomain.com" -TunnelType "L2tp" -EncryptionLevel "Required" -AuthenticationMethod MSChapv2  -L2tpPsk "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -Force -RememberCredential -DnsSuffix "MYDOMAIN.NET" -PassThru;}") 1>nul 2>nul
echo;Done!
echo;

echo;***Step 3:Clean key in regedit***
(powershell -Command "& {Remove-ItemProperty HKLM:\System\CurrentControlSet\Services\Rasman\Parameters -name "NegotiateDH2048_AES256"; Remove-ItemProperty HKLM:\System\CurrentControlSet\Services\PolicyAgent -name "AssumeUDPEncapsulationContextOnSendRule ";}") 1>nul 2>nul
echo;Done!
echo;
echo;***Step 4:Add key in Regedit***
(powershell -Command "& {New-ItemProperty HKLM:\System\CurrentControlSet\Services\Rasman\Parameters -name "NegotiateDH2048_AES256" -value 1 -propertyType dword; New-ItemProperty HKLM:\System\CurrentControlSet\Services\PolicyAgent -name "AssumeUDPEncapsulationContextOnSendRule " -value 2 -propertyType dword;}") 1>nul 2>nul
echo;Done!
echo;
echo;***Step 5:Set regedit***
powershell -Command "& {Set-ItemProperty HKLM:\System\CurrentControlSet\Services\Rasman\Parameters -name "NegotiateDH2048_AES256" -value 1; Set-ItemProperty HKLM:\System\CurrentControlSet\Services\PolicyAgent -name "AssumeUDPEncapsulationContextOnSendRule " -value 2;}"
echo;Done!
echo;
echo;***Step 6:Reboot your Windows***
echo;Press any key to reboot your windows, or press X to quit the script and manually reboot your windows.
pause>nul
goto :end

:u
cls
echo;Uninstall MY-VPN(L2TP/IPSEC VPN)
echo;===========================
(powershell -Command "& {Remove-VpnConnection -Name "MY-VPN" -Force -PassThru;}") 1>nul 2>nul
echo;Done, Press any key to quit...
goto :end



:end
pause>nul


```

