---
title: "How to Init Local Project and Push to Git Server"
date: 2021-04-01T19:23:10+08:00
lastmode: 2021-04-01T19:23:10+08:00
draft: false
tags: [ "devops", "git" ]
categories: ["devops"]
reward: true
mathjax: true
---

# How to init Local project and push it to git server

  1. Create local project
  
  ```bash 
  git init
  ```
  
  2. Create a repositories on git server
 
  ```bash
  https://mygitserver/my/myproject.git
  ```
  
  3. Add remote origin

  ```bash
  git remote add origin https://mygitserver/my/myproject.git
  ```
  
  4. Pull remote master branche and merge it with local master branche.

  ```bash
  git pull origin master:master
  ```
  
  5. Push local branch to remote

  ```bash
  git push -u origin master
  ```
  
  
  6. Add local project and push it on git server
  
  ```bash
  git add -A
  git commit -m "Inistal at first"
  git push --set-upstream origin master
  ```

