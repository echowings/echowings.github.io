---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
image: https://picsum.photos/800/600.webp?random={{ substr (md5 (.Date)) 4 8 }}
categories:
  - Blog
tags:
  - Hugo
slug:
  - test
layout: "post"
slug: "post"
license: CC BY-NC-ND
---

