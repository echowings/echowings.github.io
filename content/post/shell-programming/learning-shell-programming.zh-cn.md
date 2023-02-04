---
title: "Shell编程学习 01"
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

# 01.shell

## Shell 脚本的建立和执行

1. 脚本第一行指定脚本的解释器

```bash
#!/bin/bash
```

2. `source script-name` 或者"."读入或加载知道那个的 shell 脚本文件

3. shell 脚本的变量继承关系
   使用 shell 脚本来加载脚本中的变量，使用 source ./file 或者`. ./file`来加载。直接使用 sh 来执行是无法加载的。原因是 sh file 启动了新的 shell 来执行脚本，当执行结束，新的 shell 退出导致变量无法被继承下来。使用 source 或者.来运行程序，是在当前 shell 中运行，所以脚本中的变量是可以被继承下来的。

## shell 编程归还

1. shell 脚本的第一行指定脚本解释器

```bash
#!/bin/bash
```

或者

```bash
#!/bin/sh
```

2. shell 脚本开头加上版本、版权等信息：

```bash
# Date: 17:25 CST 2023-2-3
# Author: Create by Steve Dong
# Blog: https://echowings.github.io
# Description: This scriptes funciton is ...
# Version 1.0
```

3. 环境变量初始化与对应文件的生效顺序

  - 通过系统用户登录后默认运行的 shell

{{< mermaid align="left" theme="neutral" >}}
graph TD
	A[ /etc/profile ] -->  B( /etc/profile.d ) 
	B --> C[ $HOME/.bash_profile ] 
	C --> D[ $HOME/.bashrc ] 
	D --> E[ /etc/bashrc ]
{{< /mermaid >}}

  -   非登录交互运行的 shell
  -   执行脚本运行非交互式 shell
{{< mermaid align="left" theme="neutral" >}}
graph TD
A[ $HOME/.bashrc ] --> B[ /etc/bashrc ]
{{< /mermaid >}}
