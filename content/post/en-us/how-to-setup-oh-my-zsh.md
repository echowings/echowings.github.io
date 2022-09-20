---
title: "How to Setup Oh My Zsh"
date: 2019-04-10T12:18:04+08:00
tags: ['zsh']
categories: ['zsh']
author: 'steve dong'
draft: false
reward: false
---

# How to setup Oh-My-Zsh


## Install support application 

```shell
apt install git curl wget zsh
```

## Install zsh

  - via curl

  ```shell
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
  ```
  - via wget

  ```shell
  sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
  ```



## Change zsh settings

- change theme

```shell
#change theme froy robbyrussell to agnoster
sed -i 's/ZSH_THEME=\"robbyrussell\"/ZSH_THEME=\"agnoster\"/g' ~/.zshrc
```

- change plugin support


```shell
sed -i 's/plugins=(git)/plugin=(\n\tgit\n\tboundler\n\tdotenv\n\trake\n\trbenv\n\truby\n\tpython\n)/g' ~/.zshrc
```

## Switch default bash to zsh

```
chsh -s /bin/zsh
```

