---
title: "Rime Settings"
date: 2024-02-05T16:56:13+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=fd5d93e8
categories:
  - inputmethod
tags:
  - rime
  - win
  - mac
  - linux
slug:
  - rime
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

## Git Repo

[rime-setting](https://github.com/Iorest/rime-setting)



## Install rime /squirrel on mac/windows/linux



### Windows

Install [RIME](https://rime.im/)

### MacOS
```shell
brew install squirrel
```
### Linux

## Clone git repo to local

```shell
git clone https://github.com/Iorest/rime-setting.git
```

## Install configuration
### Install configuration on windows

copy all files except `font` folder into
`%APPDATA%\Rime`

### Install configuration on macos

Copy all files except `font` folder inot
`~/Library/Rime`

## change `default.custom.yaml`

```yaml
# add the command to command to page up,  perido for page down
  - { when: has_menu, accept: "comma", send: Page_Up }
  - { when: has_menu, accept: "period", send: Page_Down }
```

## Change fonts in file `weasel.custom.yaml`

```yaml
  "style/font_face": "LXGW WenKai GB Screen R"
```


## Click redeploy on rime
