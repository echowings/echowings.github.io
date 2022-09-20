---
title: "How to Install Hugo With Loveit Theme"
date: 2022-04-01T18:15:20+08:00
lastmode: 2022-04-01T18:15:20+08:00
draft: false
tags: [ "github pag", "git_action","loveit" ]
categories: ["DevOps"]
reward: true
mathjax: true
---

**You can learn how to build hugo blog with loveit theme. With github action's help you can auto deploy to github page**

## Install Hugo



## Install theme and plugin

```bash
git init
git remote add origin git@github.com:echowings/blogs.git


git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
git submodule add https://github.com/kaushalmodi/hugo-search-fuse-js.git themes/hugo-search-fuse-js

git submodule add --force -b master git@github.com:echowings/echowings.github.io.git public  
```

```markup
{{</* mermaid */>}}
GRAPH LR;
A-->B
```
