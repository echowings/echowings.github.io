---
title: "How to Setup Vim"
date: 2019-04-09T10:02:33+08:00
tags: ["vim"]
categories: ["vim"]
auther: "steve dong"
draft: false
reward: true
---
# How to setup vim


## Install vundle
vundle is a plugin manage tools for vim

```bash

git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim

```

## Edit vim configure file

```shell
cat > ~/.vimrc <<EOF
set nocompatible              " be iMproved, required
filetype off                  " required
syntax on
" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')
" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'
" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
" Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
" Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Install L9 and avoid a Naming conflict if you've already installed a
" different version somewhere else.
Plugin 'ascenator/L9', {'name': 'newL9'}
Plugin 'morhetz/gruvbox'
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'
" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
EOF
```
## Install Plugin
```shell
vim
```
Enter `:PluginInstall` then wait for the installation process finished.
then run the commands:
```vim
cat >> ~/.vimrc <<EOF
colorscheme gruvbox
set background=dark    " Setting dark mode
let g:gruvbox_contrast_dark='hard'
let g:airline_theme='simple'
EOF
```

## Let vim support nginx syntax highlight

```shell 
mkdir -p ~/.vim/syntax/
wget -O ~/.vim/syntax/nginx.vim http://www.vim.org/scripts/download_script.php?src_id=14376
# /usr/local/etc/nginx/vhosts/* change it as you wish
echo "au BufRead,BufNewFile /usr/local/etc/nginx/vhosts/* set ft=nginx" >> ~/.vim/filetype.vim
```

## Let vim remember last opened position

```shell
cat >> ~/.vimrc << EOF
" Uncomment the following to have Vim jump to the last position when                                                       
" reopening a file
if has("autocmd")
  au BufReadPost * if line("'\"") > 0 && line("'\"") <= line("$")
    \| exe "normal! g'\"" | endif
endif
EOF
```
# Change tab to 4 space

```shell
cat >> ~/.vimrc <<EOF
set tabstop=4
set shiftwidth=4
set softtabstop=4
set expandtab
EOF
```

## Reference

- [让Vim支持Nginx语法高亮 -Ubuntu](http://aleonchen.com/2017/02/06/vim/)
- [In vim, how do I get a file to open at the same line number I closed it at last time?](https://stackoverflow.com/questions/774560/in-vim-how-do-i-get-a-file-to-open-at-the-same-line-number-i-closed-it-at-last)
- [How do I change tab size in Vim?](https://stackoverflow.com/questions/2054627/how-do-i-change-tab-size-in-vim)


