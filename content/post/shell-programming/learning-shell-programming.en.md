---
title: "Learning Shell Programming 1"
date: 2023-02-04T08:43:35+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=bcffb920
categories:
  - shell
tags:
  - shell
  - bash
slug:
  - shell
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


# 01.Shell programming

## Shell create and running

1. The first line of the shell script is to set the shell interpter.
```bash
#!/bin/bash
```

2. `source script-name` or "." is to read or load which shell variable in the shell script.

3. sequeues to load variables in the shell scripts .
When we use command `source ./file` or `. file` to load varialbe. the shell will excuted in the same shell as we just run it in the same shell.

when we run `sh shell_script.sh`, it will open a new shell interpter to run it. when the shell excute end. the shell will ended. and the variable will not be read into current we excuted.

## shell tips

1. Indicator the shell interpter at the 1st line of the shell script

```bash
#!/bin/bash
```
or

```bash
#!/bin/sh
```

2. shell Add more informaton in the comments
```bash
# Date: 17:25 CST 2023-2-3
# Author: Create by Steve Dong
# Blog: https://echowings.github.io
# Description: This scriptes funciton is ...
# Version 1.0
```


## environment variable initial and loading config file sequques

  - system user login the shell

{{< mermaid align="left" theme="neutral" >}}
graph TD
	A[ /etc/profile ] -->  B( /etc/profile.d ) 
	B --> C[ $HOME/.bash_profile ] 
	C --> D[ $HOME/.bashrc ] 
	D --> E[ /etc/bashrc ]
{{< /mermaid >}}

  -   nolgin interactive shell
  -   runing script   without interactive shell

{{< mermaid align="left" theme="neutral" >}}
graph TD
A[ $HOME/.bashrc ] --> B[ /etc/bashrc ]
{{< /mermaid >}}
