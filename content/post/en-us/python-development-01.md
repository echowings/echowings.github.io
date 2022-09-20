---
title: "Python Development 01"
date: 2018-11-29T00:28:31+08:00
lastmod: 2018-12-24T16:01:23+08:00
tags: ["python"]
categories: ["programming"]
author: "Steve Dong"
reward: true
---

# Python Development

## python开发笔记-01-python开发入门
### 安装ptyhon开发工具和包
### 安装ubuntu 16.04

...pass...

### 安装系统相关依赖包
build-essential包可以批量安装Python在Ubuntu上进行构建时所需的全部工具(gcc,make等).

```bash
sudo apt-get -y update && \
  sudo apt-get -y upgrade &&  \
  sudo apt-get install -y build-essential \
  libsqlite3-dev \
  libreadline6-dev \
  libgdbm-dev \
  zlib1g-dev \ 
  libbz2-dev \ 
  sqlite3 \
  tk-dev \
  zip \
  libssl-dev
```

###  安装python相关依赖包

install python-dev和pip

``` bash
# Install python-dev
sudo apt-get -y install python-dev

#install pip (before python 2.7.9)
wget https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py


#在python 3.4和python2.7.9之后加入了 ensurepip模块,也就是安装python的时候默认捆绑安装pip,更新pip使用
#python -m ensurepip -U
sudo python -m pip install --upgrade pip
```

##使用get-pip.py来安装最新版本的pip

* 查看python版本

``` bash
$python -V
Python 2.7.12
```

* 查看pip版本

```bash
$ pip --version
pip 18.1 from /usr/local/lib/python2.7/dist-packages/pip (python 2.7)
```

* 通过pip安装开发用的软件
* 安装virtualenv包

``` bash
$sudo pip insetall virtualenv
Collecting virtualenv
  Downloading https://files.pythonhosted.org/packages/7c/17/9b7b6cddfd255388b58c61e25b091047f6814183e1d63741c8df8dcd65a2/virtualenv-16.1.0-py2.py3-none-any.whl (1.9MB)
    100% |################################| 1.9MB 2.5MB/s
Installing collected packages: virtualenv
Successfully installed virtualenv-16.1.0
virtualenv 的使用方法 virtualenv是搭建虚拟的python执行环境.
  $ virtualenv --version
  16.1.0
```

* 查看当前pip 安装的软件

```bash
  $ pip freeze 
  ansible==2.7.2
asn1crypto==0.24.0
bcrypt==3.1.4
cffi==1.11.5
cryptography==2.4.2
enum34==1.1.6
idna==2.7
ipaddress==1.0.22
Jinja2==2.10
MarkupSafe==1.1.0
paramiko==2.4.2
pyasn1==0.4.4
pycparser==2.19
PyNaCl==1.3.0
PyYAML==3.13
six==1.11.0
virtualenv==16.1.0
```

* 搭建virtualenv环境
 
```bash
 $mkdir ~/work
 $cd ~/work
 $virtualenv venv
 New python executable in /home/user1/work/venv/bin/python
Installing setuptools, pip, wheel...
done. 
```
* 启动virtualenv 环境

```bash
 $source venv/bin/activate
 (venv) user1@python-dev-env:~/work$
#使用pip freeze查看已经安装pip包为空
 $pip freeze
``` 

 
- 关闭virtualenv环境

``` bash
$ deactivate
```

- 删除virtualenv

```bash
rm -R vienv
```

### 多python版本使用

``` bash
user1@python-dev-env:~/work$ python3 -V
Python 3.5.2
user1@python-dev-env:~/work$ python2 -V
Python 2.7.12   
使用virtualenv 设置不同版本的python 
mkdir -p ~/work/venv-python2
mkdir -p ~/work/venv-python3
cd ~/work
virtualenv --python=/usr/bin/python2 venv-python2
virtualenv --python=/usr/bin/python3 venv-python3
source venv-python2/bin/activate
python -V
pip freeze
deactivate
source venv-python3/bin/activate
python -V
deactivate
```
   
### Mercurial 为什么使用 mecurial, python编写.易于上手.内置web管理,支持bitbucket,用TortoiseHg等GUI客户端. - 安装Mercurial

``` bash
$sudo pip install mercurial
Collecting mercurial
Downloading https://files.pythonhosted.org/packages/6a/f4/4c3d3a2bf950f3261f506284952e23cee2b62e50eb8974aa961228a95e8a/mercurial-4.8.tar.gz (6.9MB) 100% |################################| 6.9MB 1.3MB/s Installing collected packages: mercurial Running setup.py install for mercurial … done Successfully installed mercurial-4.8 

```

- 安装zsh

```bash
sudo apt-get install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"  
# vi ~/.zhsrc
plugins=(
git bundler dotenv osx rake rbenv ruby mercurial )

ZSH_THEME=“agnoster” 

```

- 查看Mercurial版本

```bash
   user1@python-dev-env:~$ hg --version
	Mercurial Distributed SCM (version 4.8)
	(see https://mercurial-scm.org for more information)

	Copyright (C) 2005-2018 Matt Mackall and others
	This is free software; see the source for copying conditions. There is NO
	warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
	
- 创建版本库

1. 创建Mercurial的环境设置文件.在主目录下创建.hgrc文件, 括号内的称为节(Section),如[ui]用来设置usernme的属性.username会想版本哭提交时候记录在日志里
2. [Extensions]用来激活Mercurial附属的扩展工具,将扩展工具名写入[extensions],我们就可以使用该工具了.有一些工具不但要在Extension中设置,还要在pager中设置pager属性

```bash
vi ~/.hgrc
```

`[ui] username=bpbook bpbook@beproud.jp [extensions] color= pager= [pager] pager=LESS=‘FSRX’ less`


```python
  mercurial
  mkdir ~/hgtest
  cd ~/hgtest
  hg init
  touch test.txt
  hg status
  ? test.txt
  
  # ？代表没有添加到hg库里、M标示修改, A标示有添加
  
  # 添加文件　 hg  add
  hg add test.txt
  hg status
  A test.txt
  
  # 提交文件
  $hg commit
  test commit
  #在打开的编辑器中,填写commit,然后保存退出,完成提交
  
  修改~/.hgrc , [ui]节添加
  [ui]
  editor = vim
  
  $ echo $EDITOR
  vim
  
  # hg staus 查看提交后的状态
  hg status
  
  #查看差别
  $ hg diff 
  
  # 撤销编辑(未提交的内容)
  $ hg revert test.txt
  
  # 撤销已经提交的内容
  $ hg commit --amend
```

  
  

### python代码编辑器

- vim

``` vim
" 将“Tab”替换为“空格”
setl expandtab
" 将“Tab”的“缩进幅度”改为 4
setl tabstop=4 
“ 自动缩进时的“缩进幅度”改为 4 
setl shiftwidth=4 
“ 按下键盘“Tab”键时插入的空格数 “ 这里设置为 0 就可以插入“tabstop”中设置的空格数了 
setl softtabstop=0 
“ 保存时删除行尾的空格 
autocmd BufWritePre * :%s/\s+$//ge 
“80 个字符换行 setlocal textwidth=80

```

 - install neobundle.vim

```bash
 cd ~
 curl https://raw.githubusercontent.com/Shougo/neobundle.vim/master/bin/install.sh | sh
 vim ~/.vimrc
 NeoBundle 'nvie/vim-flake8'
```

* pbd调试

```bash
   	def add(x, y):
   	    return x + y
	x= 0
	import pdb; pdb.set_trace() x = add(1, 2)

$python pdbtest.py
    
> pdbtest.py(7)<module>()
  -> x = add(1, 2)
  (Pdb)
```