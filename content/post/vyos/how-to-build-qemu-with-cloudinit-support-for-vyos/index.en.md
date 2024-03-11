---
title: "How to Build Qemu With Cloudinit Support for Vyos"
date: 2024-02-21T16:17:56+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=d09996cd
categories:
  - vyos
  - pve
  - cloudinit
tags:
  - vyos
  - qemu
  - cloudinit
  - build
  - docker
  - qcow2
slug:
  - vyos
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

## Perepare Linux
For me I perfer to Debian linux

## Install docker
  - Install docker
    ```shell
    apt install -y wget curl 
    cp /etc/apt/sources.list /etc/apt/sources.list-bak
    wget https://mirrors.ustc.edu.cn/repogen/conf/debian-https-4-bullseye -O /etc/apt/sources.list
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io
    sudo systemctl start docker
    sudo systemctl enable docker
    ```

## Install software
  - Install git
    ```shell
    apt install -y git-core
    ```


## Git Repository

  - git pull 
    ```shell
    git clone https://github.com/vyos/vyos-vm-images.git
    cd vyos-vm-images
    ```

  - commit download-iso on qemu.yaml
      - Edit `qemu.yml`
        ```shell
        vi qemu.yml
        ```
      - commit `- download-iso`
        ```yaml
        - install-packages
        # - download-iso
        - mount-iso
        ```
  - Change vyos_qemu_img path from `/tmp` to `tmp`
    ```shell
    vi qemu.yml 
    ```
    Change
    ```yaml
    vyos_qemu_img: "/tmp/vyos-{{ vyos_version }}{{ ci_tag  | default() }}-{{vyos_disk_size}}G-qemu.qcow2"
    ```
    to:
    ```yaml
    vyos_qemu_img: "tmp/vyos-{{ vyos_version }}{{ ci_tag  | default() }}-{{vyos_disk_size}}G-qemu.qcow2"
    ```

## Download vyos iso file
  - Build or copy iso image to local
    ```shell
    mkdir tmp
    curl -fsSL https://xxxxxxx/vyos-1.3.6-amd64.iso -o tmp/vyos-1.3.6-amd64.iso
    ls /tmp/*.iso
    ```


## Copy customized files into
  - Copy file into files
    ```bash
    make

    ```

## Build qcow2 image
  - Run docker
    ```shell
    #Downlaod dockerfile and run build environment in docker 
    wget https://raw.githubusercontent.com/vyos/vyos-vm-images/current/Dockerfile
    # Build docker iamges
    docker build --tag vyos-vm-images:latest -f ./Dockerfile .
    ```
    Run docker
    ```bash
    docker run --rm -it --privileged -v $(pwd):/vm-build -v $(pwd)/images:/images -w /vm-build vyos-vm-images:latest bash
    ```

  - Build qcow2 image
    ```shell
    ansible-playbook qemu.yml   \
      -e disk_size=2   \
      -e iso_local=tmp/vyos-1.3.6-amd64.iso    \
      -e grub_console=serial   \
      -e cloud_init=true    \
      -e cloud_init_ds=NoCloud    \
      -e guest_agent=qemu    \
      -e enable_ssh=true
      ```
## Build image with custome files
### copy qcow2 file to `files/custom_files`

`opt/vyatta/etc/config` stand for /config folder

### Add file with `x` excute permission
  - Edit `roles/copy-custom-files/tasks/main.yml` add `mode: a+x`
    ```shell
    - name: Copy custom files to the system, preserving paths
      become: true
      copy:
        src: "files/custom_files/"
        dest: "{{ vyos_install_root }}"
        force: true
        mode: a+x
      when: custom_files is defined
    ```
  - Build qcow2 image with custom_files option

    ```shell
    # 1.3.6
    ansible-playbook qemu.yml     \
      -e disk_size=2    \
      -e iso_local=tmp/vyos-1.3.6-amd64.iso    \
      -e grub_console=serial    \
      -e cloud_init=true    \
      -e cloud_init_ds=NoCloud    \
      -e guest_agent=qemu    \
      -e enable_ssh=true \
      -e custom_files=true

    ```

    ```shell
    # 1.4.0-eap1
    ansible-playbook qemu.yml     \
      -e disk_size=2    \
      -e iso_local=tmp/vyos-1.4.0-epa1-amd64.iso    \
      -e grub_console=serial    \
      -e cloud_init=true    \
      -e cloud_init_ds=NoCloud    \
      -e guest_agent=qemu    \
      -e enable_ssh=true \
      -e custom_files=true

    ```



## Reference
  - [VyOS Qemu Image Build](https://codingpackets.com/blog/vyos-qemu-image-build/)