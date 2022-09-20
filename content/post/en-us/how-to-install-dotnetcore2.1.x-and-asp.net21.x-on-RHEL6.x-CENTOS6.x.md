---
title: "How to Install Dotnetcore2.1.x and Asp.net21.x on RHEL6.x CENTOS6"
date: 2018-10-26T22:29:04+08:00
tags: [ "dotnetcore", "linux" ]
categories: ["develop"]
author: "steve dong"
draft: false
reward: true
---


## How to install netcore/aspnetcore 2.1.4 on RHEL 6.8
Recently days, we met a scenario that is our client production environment must be RHEL 6.8. But our product is base asp.net core 2.1.x version. Rehat and Microsoft has never say the are support RHEL 6.X version.

After a few days research, I found that .net core website is support RHEL 6.X,but no runtime.We can download install SDK to support asp.net core. And you know what? It works well finally!

### Install netcore 2.1.4 sdk

```shell
#!/bin/sh

yum install -y wget epel-release
yum install -y openssl-devel libnghttp2-devel libidn-devel gcc libtool m4 automake
cd /tmp/

curl -O https://curl.haxx.se/download/curl-7.45.0.tar.gz
tar -xf curl-7.45.0.tar.gz
cd curl-7.45.0
./configure \
    --disable-dict \
    --disable-file \
    --disable-ftp \
    --disable-gopher \
    --disable-imap \
    --disable-ldap \
    --disable-ldaps \
    --disable-libcurl-option \
    --disable-manual \
    --disable-pop3 \
    --disable-rtsp \
    --disable-smb \
    --disable-smtp \
    --disable-telnet \
    --disable-tftp \
    --enable-ipv6 \
    --enable-optimize \
    --enable-symbol-hiding \
    --with-ca-bundle=/etc/pki/tls/certs/ca-bundle.crt \
    --with-nghttp2 \
    --with-gssapi \
    --with-ssl \
    --without-librtmp \
    --prefix=$PWD/install/usr/local
    
```

### make install

``` bash
cd install
tar -czf curl-7_45_0-RHEL6-x64.tgz *
curl -O http://download.icu-project.org/files/icu4c/57.1/icu4c-57_1-RHEL6-x64.tgz
tar -xf curl-7_45_0-RHEL6-x64.tgz -C /
tar -xf icu4c-57_1-RHEL6-x64.tgz -C /
echo "export LD_LIBRARY_PATH=/usr/local/lib" >> /etc/profile
export LD_LIBRARY_PATH=/usr/local/lib

cd /tmp/
mkdir dotnetsdk

curl -O https://download.microsoft.com/download/8/A/7/8A765126-50CA-4C6F-890B-19AE47961E4B/dotnet-sdk-2.1.402-rhel.6-x64.tar.gz

tar -zxvf dotnet-sdk-2.1.402-rhel.6-x64.tar.gz -C dotnetsdk/

mv dotnetsdk /opt/
ln -s /opt/dotnetsdk/dotnet /usr/bin/dotnet

curl -sSL https://github.com/libuv/libuv/archive/v1.4.2.tar.gz | sudo tar zxfv - -C /usr/local/src


# Insall libuv-1.4.2 
cd /usr/local/src/libuv-1.4.2
sh autogen.sh
./configure
make
make install
rm -rf /usr/local/src/libuv-1.4.2 && cd ~/
ldconfig

```

## Reference
* [How to use .NET Core on RHEL 6 / CentOS 6](https://github.com/dotnet/core/blob/master/Documentation/build-and-install-rhel6-prerequisites.md)

* [.NET Core 2.1.4](https://github.com/dotnet/core/blob/master/release-notes/2.1/2.1.4/2.1.4-download.md)


