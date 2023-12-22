---
title: "How to Build Vyos Image 1.3.5"
date: 2023-12-22T17:41:27+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=9b164b10
categories:
  - Blog
tags:
  - vyos
slug:
  - vyos
layout: 
  - vyos
slug: 
  - vyos

license: CC BY-NC-ND
---
```shell
sudo apt update 
sudo apt install -y unzip

VYOS_VERSION="1.3.5"
YOUR_MAIL="MYMAIL@EXAMPLE.COM"
curl -O -L https://github.com/vyos/vyos-build/archive/refs/tags/$VYOS_VERSION.zip
unzip $VYOS_VERSION.zip
cd vyos-build-$VYOS_VERSION

docker build -t vyos-builder docker


docker run --rm -it --privileged -v $(pwd):/vyos -w /vyos vyos-builder bash

VYOS_VERSION="1.3.5"
YOUR_MAIL="MYMAIL@EXAMPLE.COM"

# vi scripts/packer.json
sed -i '/"disk_size/c\"disk_size":\ 2048,' scripts/packer.json
cat scripts/packer.json | grep disk_size

#sed -i '3 i New Line with sed' File1
sed -i '51 i        "set system console device ttyS0 speed 115200<enter><wait5>",' scripts/packer.json
cat scripts/packer.json | grep tty

sed -i '/IMAGE_SIZE/c\IMAGE_SIZE=2' scripts/build-azure-image
cat scripts/build-azure-image | grep IMAGE_SIZE

./configure --architecture amd64 --build-by "$YOUR_MAIL" --build-type release --version $VYOS_VERSION

sudo make iso

sudo packer plugins install github.com/hashicorp/qemu

sudo make qemu
```