---
title: "How to Install and Configure Zsh and Zim"
date: 2022-12-01T12:55:48+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=708fb2d3
categories:
  - zsh
tags:
  - zsh
  - macos
  - linux
  - debian
slug:
  - zsh
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

# How to install zsh zim on macos & debian

## Install and configure zsh zim iterm2 on macos

### Install homebrew

```bash
/bin/bash -c "$(curl -fsSL <https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh>)"
```

### Install iterm2

```bash
brew install --cask iterm2
```

### Install zsh

```bash
brew install zsh
```

### Install zim

```bash
curl -fsSL <https://raw.githubusercontent.com/zimfw/install/master/install.zsh> | zsh
```

### Install Powerlevel10k

Add `.zimrc`

```txt
zmodule romkatv/powerlevel10k
```

Change configure with shell script

```bash
 if [ `grep  -c  "zmodule\ romkatv\/powerlevel10k" ~/.zimrc` -ne '0' ]; then echo "exist"; else echo "not exist"&& echo zmodule\ romkatv/\powerlevel10k >> ~/.zimrc; fi
```

Install zim modules

```bash
zimfw install
```

## Install and configure zsh zim on debian 11

### Install zsh zimfw

```bash
apt install -y zsh curl

curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh
```

### Install module

#### Edit `~/.zimrc`

Add follow text into `~/.zimrc`

```bash
# Zim theme
zmodule eriner

#powerlevel10k
zmodule romkatv/powerlevel10k
```

#### Install Modules

```bash
# Install modules
zimfw install

#update modules
zimfw update


#uninstall modules
# zimfw uninstall xxxx

```

## Reference

-   [Beautiful your mac terminal, iterm2 + zim + powerlevel10k](https://rainlay.medium.com/beautiful-your-mac-terminal-iterm2-zim-powerlevel10k-2f3dedde5b85)
-   [07 - Zim - Zsh 配置框架與它的插件](https://ithelp.ithome.com.tw/articles/10270581)
