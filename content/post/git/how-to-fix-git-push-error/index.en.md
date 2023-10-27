---
title: "How to Fix Git Push Error"
date: 2023-10-27T17:25:57+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=80a2a303
categories:
  - Blog
tags:
  - git
slug:
  - git
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---
```bash
rm -rf .git/refs/remotes/origin
mkdir .git/refs/remotes/orgin
git gc --prune=now
```
## Reference
---
  - [git push 遇到 error: cannot update the ref ‘refs/remotes/origin/master‘: Permission denied](https://blog.csdn.net/weixin_43979544/article/details/121696000)