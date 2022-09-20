---
title: "How to Fix Linux Perl LC_ALL_ERROR"
date: 2019-04-16T14:36:34+08:00
tags: ["per-lc_all_error"]
categories: [ "linux", "perl" ]
author: "steve dong"
draft: false
reward: true

---

# How to fix perl LC_ALL ERROR

```shell
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8

cat >> /etc/environment <<EOF
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
EOF

```