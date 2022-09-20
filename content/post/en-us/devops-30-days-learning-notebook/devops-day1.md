---
title: "Devops Day1"
date: 2019-07-23T10:16:43+08:00
lastmode: 2019-07-23T10:16:43+08:00
draft: true
tags: [ "tag-1", "tag-2","tag-3" ]
categories: ["index"]
reward: true
mathjax: true
---

# Devops learning day1

## Install pyenv 

```bash
# Install Homebrew if it isn't already available
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 

# Install pyenv
brew install pyenv

# Add pyenv initializer to shell startup script
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
#or
echo 'eval "$(pyenv init -)"' >> ~/.zshrc

# Reload your profile
source ~/.bash_profile
#or
source ~/.bash_profile

#Install python3
pyenv instlal python3.7.4

```



##  Install pyramid

```
pip install pyramid
```

## Create a pyramid project


```bash
pcreate -s alchemy devops_2019_project
```

Use command of pyramid `pcreate` to create a project.There are three template:

|Type | Details|
|---|---|
| alchemy | Pyramid project using SQLAlchemy, SQLite, URL dispatch, and Jinja2|
|starter | Pyramid starter project using URL dispatch and Jinja2|
|zodb |  Pyramid project using ZODB, traversal, and CHameleon |


  - seting virtualenv for the project




