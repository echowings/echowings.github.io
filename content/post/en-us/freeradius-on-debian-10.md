---
title: "How to Install and Configure Freeradius With Active Directory Allow Allow Specific Group of Users to Authenticate in Debian 10"
date: 2019-10-13T14:52:31+08:00
lastmode: 2019-10-13T14:52:31+08:00
draft: false
tags: [ "freeradius", "debian","linux", "ldap" ]
categories: ["linux"]
reward: true
mathjax: true
---

{% raw %}
# How to install and configure FreeRADIUS with Active Directory allow specific group of users to authenticate in Debian 10

serval years ago,I built freeradius server in centos 6 work with active directory. It works perfect with wifi authortication  and ikev2 vpn authortication. But recently days, I found a bug that the radius server can not limit user access  to a group in AD. So I'm trying to build a new freeradius server in debian 10. After a week work.At last I figure it out.



## Install Software and configuration
First thing first, We need install a debian 10 server on your virtualization platform. currently, I installed a debian server on proxmox ve platform.
### Basic Information
|Name | Value|
|---|---|
|Domain Name| TESTING.LOCAL|
|NTDomain  Name| TESTING |
|RADIUS HOSTNAME | MYRADIUS |
|DOMAIN CONTROLLER | MYDC02.TESTING.LOCAL|
|DOMAIN CONTROLLER | MYDC03.TESTING.LOCAL|
|DOMAIN GROUP  | VPN_GROUP |
### Change Debian settings
- Change hostname 

 ```bash
hostnamectl set-hostname myradius
```
- Sync NTP time with domain controller
Install chrony

 ```bash
apt purge ntp
apt install -y  chrony
```
```bash
vi /etc/chrony/chrony.conf
#comment out `pool xxx iburst
server <ip_of_dc01> iburst
server <ip_of_dc02> iburst
```
restart chrony service and force update times
```bash
systemctl restart chrony
chronyc sources
```

### Install samba and winbind and let Debian Host joined domain
- Install samba,winbind,krb5-user

 ```bash
apt update && apt -y dist-ugprade
apt install -y samba winbind krb5-user
apt -y install winbind libpam-winbind libnss-winbind krb5-config samba-dsdb-modules samba-vfs-modules
```

- config samba config file


 ```bash
vi /etc/samba/smb.conf
```


 In `[global]` section, change settings form

 ```bash
[global]
## Browsing/Identification ###
# Change this to the workgroup/NT-domain name your Samba server will part of
   workgroup = WORKGROUP
```

 to 

  ```bash
  [global]
  ## Browsing/Identification ###
  # Change this to the workgroup/NT-domain name your Samba server will part of

     workgroup = TESTING
     realm = TESTING.LOCAL
     security = ads
     idmap config * : backend =tdb
     idmap config * : range = 3000-7999
     idmap config testing : backend = rid
     idmap config testing : range = 10000-999999
     winbind use default domain = true
     winbind offline logon = false
  ```



- Modify `/etc/nsswitch.conf`

 ```bash
  vi /etc/nsswitch.conf
  ```

Change settings:
 
  ``` bash
  passwd:         files systemd 
  group:          files systemd 
  ```


to

   ```bash
   passwd:         files systemd winbind
   group:          files systemd winbind
   ```
 
- Modify `/etc/krb5.conf`

 ```bash
[libdefaults]
    default_realm=TESTING.LOCAL
...
[realms]
    TESTING.LOCAL = {
        kdc = MYDC02.TESTING.LOCAL
        kdc = MYDC03.TESTING.LOCAL
        admin_server = TESTING.LOCAL
        default_domain = TESTING.LOCAL
    }
...
[domain_realm]
    .testing.local = TESTING.LOCAL
    testing.local = TESTING.LOCAL
```

- Restart OS


  ``` bash
  reboot
  ```


- Join domain
 
  ``` bash
  #net join -U mypoweruser
  net ads join -U Administrator
  ```

  ``` bash
  Enter Administrator's password:
  Using short domain name -- TESTING
  Joined 'SMB' to dns domain ''
  No DNS domain configured for smb. Unable to perform DNS Update.
  DNS update failed: NT_STATUS_INVALID_PARAMETER
  ```

- Restart samba and winbind service


  ```bash
  systemctl restart smbd winbind
  ```


- Testing samba AD authentication:


  * Using winbind:

   ```bash
   systemctl restart winbind
   wbinfo -a <user>%<password>
   ```
   You will get the following message if everything is correct:

  ```bash
  plaintext password authentication succeeded
  challenge/response password authentication succeeded
  ```
   * Using ntlm_auth:

   ```bash
   ntlm_auth --request-nt-key --domain=TESTING --username=<user> --password=<password>
   ```
   Then you will got success message:
   ```bash
   NT_STATUS_OK: Success (0x0)
   ```
   
  

### Install freeradius 
- Install freeradius

  ```bash
  apt install -y freeradius
  ```

- Grant permission for `freerad` user on winbind's socket:

  ```bash
  sudo usermod -a -G winbindd_priv freerad
  sudo chgrp winbindd_priv /var/lib/samba/winbindd_privileged/
  ```
- Change MACHAP to use ntlm_auth:

  ```bash
  vi /etc/freeradius/3.0/mods-available/ntlm_auth
  ```
   change 
   

  ```bash
  program = "/usr/bin/ntlm_auth --request-nt-key --domain=TESTING --username=%{mschap:User-Name} --password=%{User-Password}"
  
  #if you want to limted to a specific domain group please modified as this:
  program = "/usr/bin/ntlm_auth --request-nt-key --domain=$DOMAINNAME  --require-membership-of='$DOMAINNAME\$DOMAIN_GROUP'  --username=%{mschap:User-Name} --password=%{User-Password}"
  ```


- Change module mschap
 
```bash
vi /etc/freeradius/3.0/mods-available/mschap
```
change

```bash
{% raw %}
mschap {
    ntlm_auth = "/usr/bin/ntlm_auth --request-nt-key --domain=TESTING --username=%{%{Stripped-User-Name}:-%{%{User-Name}:-None}} --challenge=%{%{mschap:Challenge}:-00} --nt-response=%{%{mschap:NT-Response}:-00}"
{% ednraw %}
```
- Change eap config.

  ```bash
  vi /etc/freeradius/3.0/mods-available/eap
  ```
  
  Change  settings of `default_eap_type`.
  
  ```bash
  eap {
      default_eap_type = md5
  ```
  to

  ```bash
  eap {
      default_eap_type = mschapv2
  ```
  Then restart freeradius service

  ```bash
  systemctl restart freeradius
  ```


- Change `/etc/freeradius/3.0/sites-enabled/default` and `/etc/freeradius/3.0/sites-enabled/inner-tunnel`

  ``` bash
  authenticate {
      ntlm_auth
  ```
 
- Configure RADIUS client

   ```bash
   vi /etc/freeradius/3.0/clients.conf
   ```
   ```bash
   client 0.0.0.0/0 {
      secret  = myauthpassword
      shortname = "nas access"
   }
   ```
   
 
 - restart service of freeradius

    ```bash
    systemctl restart freeradius
    ```
    Test FreeRADIUS and MSCHAP:

  ```bash
radtest -t mschap <user> <password> localhost 0 testing123
```

 The results will be like:

 ```bash
Sent Access-Request Id 9 from 0.0.0.0:59244 to 127.0.0.1:1812 length 134
        User-Name = "user"
        MS-CHAP-Password = "password"
        NAS-IP-Address = 172.100.99.100
        NAS-Port = 0
        Message-Authenticator = 0x00
        Cleartext-Password = "password"
        MS-CHAP-Challenge = 0x163bc4c900360a08
        MS-CHAP-Response = 0x0001000000000000000000000000000000000000000000000000382764ceb05312077d21d71bf53ce917ef2e72a4ff83ca96
Received Access-Accept Id 9 from 127.0.0.1:1812 to 0.0.0.0:0 length 84
        MS-CHAP-MPPE-Keys = 0x000000000000000065c53b0540ab884edc6779a1f370c0cb
        MS-MPPE-Encryption-Policy = Encryption-Allowed
        MS-MPPE-Encryption-Types = RC4-40or128-bit-Allowed
```
 
### Configure freeradius-ldap Auth with AD

To limited to auth a AD group, we need to config freeradius auth with ldap.
- Install freeradius-ldap

```bash
apt install -y freeradius-ldap
```

#### Edit config
-  edit `/etc/freeradius/3.0/mods-available/ldap`

 ```bash
vi /etc/freeradius/3.0/mods-available/ldap
```
```bash
ldap {
     ...
    server = 'dc01.mydomain.local'
    server = 'dc02.mydomain.local'
    port = 389
    identity = 'aduser@mydomain.local'
    password = <mypassword>
    base_dn = 'dc=mydomain,dc=local'
    ... 
}
...
    update {
       control:Password-With-Header    += 'userPassword'
       control:NT-Password     := 'ntPassword'
       reply:Reply-Message     := 'radiusReplyMessage'
       reply:Tunnel-Type       := 'radiusTunnelType'
       reply:Tunnel-Medium-Type    := 'radiusTunnelMediumType'
       reply:Tunnel-Private-Group-ID   := 'radiusTunnelPrivategroupId'

        #  Where only a list is specified as the RADIUS attribute,
        #  the value of the LDAP attribute is parsed as a valuepair
        #  in the same format as the 'valuepair_attribute' (above).
        control:            += 'radiusControlAttribute'
        request:            += 'radiusRequestAttribute'
        reply:              += 'radiusReplyAttribute'
    }
...
edir = no
...
}
...
user {
...
    base_dn = "${..base_dn}"
...
    filter = "(sAMAccountName=%{%{Stripped-User-Name}:-%{User-Name}})"    
...
```

  - create a link to mode-enable

  ```bash
  ln -s /etc/freeradius/3.0/mods-available/ldap /etc/freeradius/3.0/mods-enabled/
  ```

 -  edit `/etc/freeradius/3.0/sites-available/default`

 ```bash
 vi /etc/freeradius/3.0/sites-available/default
 ```
 
 ```bash
...
authorize {
...
    ldap
    if ((ok || updated) && User-Password) {
        update {
            control:Auth-Type := ldap
        }
    }
...
authenticate {
...
        Auth-Type LDAP {
                ldap
        }
...
}
...
}
```


- Restart freeradius service

 ```bash
 systemctl restart freeradius
 freeradius -fX
 ```

  testing ldap auth

  ```bash
  radtest <domain_accout> <password> localhost 0 testing123
  ```

 
-  Change `/etc/freeradius/3.0/users` to allow specific group `VPN_GROUP` of users to authenticate

 ```bash
 vi /etc/freeradius/3.0/users
 ```

 ```bash
 DEFAULT Auth-Type = ntlm_auth, LDAP-Group == "VPN_GROUP"
 ...
 DEFAULT Group != "VPN_GROUP", Auth-Type := Reject
         Reply-Message = "Your are not permit to access VPN Connectiong"
 ```
 
 - Change MACHAP to use ntlm_auth:

  ```bash
  vi /etc/freeradius/3.0/mods-available/ntlm_auth
  ```
   change 
   
  ```bash
  #if you want to limted to a specific domain group please modified as this:
  program = "/usr/bin/ntlm_auth --request-nt-key --domain=$DOMAINNAME  --require-membership-of='$DOMAINNAME\$DOMAIN_GROUP'  --username=%{mschap:User-Name} --password=%{User-Password}"
  ```


## Install mysql,daloradius to make management freeradius with web access

  - Install mariadb database

  ```bash
  apt install -y mariadb-server
  ```
  
  
## Renew certficiation 
We purchased godaddy certification, so we will replace ssl certification

- backup default eap configure file

  ```bash
  cp /etc/freeradius/3.0/mods-enabled/eap /etc/freeradius/3.0/mods-enabled/eap.backup
 
  ```
- Place pem file to `/etc/freeradius/3.0/certs/mycerts`
  
  ```bash
  ls -al /etc/freeradius/3.0/certs/mycerts
-rw-r--r-- 1 root    root    1728 Feb 21 11:11 mycerts-ca.pem
-rw-r--r-- 1 root    root    1704 Feb 21 11:09 mycerts.key
-rw-r--r-- 1 root    root    2248 Feb 21 11:13 mycerts.pem
  ```
- Convert crt to pem

 ```bash
 openssl x509 -in mycert-ca.crt -out mycert-ca.pem -outform PEM
 ```

- Modified `/etc/freeradius/3.0/mods-enabled/eap` file.
 
  ```bash
    tls-config tls-common {
        #private_key_password = whatever
        private_key_file = /etc/freeradius/3.0/certs/vpn.grapecity.com.cn/mycerts.key
...
        certificate_file = /etc/freeradius/3.0/certs/vpn.grapecity.com.cn/mycerts.pem
...
        ca-file = /etc/freeradius/3.0/certs/vpn.grapecity.com.cn/mycerts-ca.pem
  
  ```

- Restart freeradius service.

  ```bash
  systemctl restart freeradius
  ```

{% endraw %}
## Reference 

1. [WPA2 Enterprise with FreeRADIUS and AD integration on Ubuntu16.04](https://www.dangtrinh.com/2017/03/wpa2-enterprise-with-freeradius-and-ad.html)
2. [调试freeradius 3.0 与microsoft AD通过LDAP认证的笔记](https://blog.csdn.net/d9394952/article/details/93597621)
3. [Configure FreeRADIUS with Active Driectory allow specific group of users to authenticate](https://serverfault.com/questions/622205/configure-freeradius-with-active-driectory-allow-specific-group-of-users-to-auth)
4. [Join in Windows Active Directory Domain with Samba Winbind.](https://www.server-world.info/en/note?os=Debian_10&p=samba&f=3)
5. [802.1x PEAP mschapv2认证证书问题](https://bbs.ui.com.cn/t/802-1x-peap-mschapv2/48063/5)
6. [How to convert .crt to .pem [duplicate]](https://stackoverflow.com/questions/4691699/how-to-convert-crt-to-pem)
