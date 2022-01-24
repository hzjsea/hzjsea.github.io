---
title: 使用neovim
categories:
  - linux
tags:
  - linux
abbrlink: 8d547e43
date: 2020-03-26 16:46:20
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [使用neovim](#使用neovim)
  - [安装vim](#安装vim)
  - [安装neovim](#安装neovim)
  - [安装vim-plugin](#安装vim-plugin)
  - [安装插件](#安装插件)
  - [安装coc-nvim插件](#安装coc-nvim插件)
  - [使用语言配置文件](#使用语言配置文件)
  - [插件安装](#插件安装)
  - [安装Python3环境依赖](#安装python3环境依赖)

<!-- /code_chunk_output -->
<!-- more -->

# 使用neovim

## 安装vim
```bash
mkdir /root/package/
cd /root/package/
# 安装环境
yum install ruby ruby-devel lua lua-devel luajit \
luajit-devel ctags git python python-devel \
python36 python36-devel tcl-devel \
perl perl-devel perl-Extutils-ParseXS \
perl-ExtUtils-XSpp perl-ExtUtils-CBuilder \
perl-ExtUtils-Embed libX* ncurses-devel gtk2-devel \
gcc

# 卸载原来不符合版本要求的version vim
yum -y remove vim

# 获取vim
git clone https://github.com/vim/vim.git

# 编译安装
cd vim && ./configure --with-features=huge \
--enable-gui=gtk2 \
--with-x \
--enable-fontset \
--enable-cscope \
--enable-multibyte \
--enable-pythoninterp \
--with-python-config-dir=/usr/lib64/python2.7/config \
--enable-python3interp \
--with-python3-config-dir=/usr/lib64/python3.6/config \
--enable-luainterp \
--enable-rubyinterp \
--enable-perlinterp \
--enable-multibyte \
--prefix=/usr/local/vim \
--with-compiledby="brooksj"

# 创建makefile
make

# 编译
make -j 8

# 设置系统环境
export PATH=/usr/local/vim/bin:$PATH 

## 注释
# 这里的路径和你vim的安装路径一样，安装路径为创建makefile文件时 指定的--prefix参数
```
## 安装neovim

```bash
yum upgrade
yum install python3 git  
cd /root/package/
curl -LO https://github.com/neovim/neovim/releases/download/stable/nvim.appimage
## 添加权限
chmod 777 ./nvim.appimage
./nvim.appimage --appimage-extract
ln -s ./squashfs-root/usr/bin/nvim /usr/bin/nvim 

mkdir -p /root/.config/  && cd /root/.config/ && touch init.vim .vimrc

# vi .bash_profile 
alias vi=/root/package/squashfs-root/usr/bin/nvim
source .bash_profile
```

## 安装vim-plugin
vim-plugin这个插件用来安装插件的，只要在command上面输入`PluginInstall`就可以了，
https://github.com/junegunn/vim-plug

```bash
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```


## 安装插件
```bash
# vi /root/.config/nvim/init.vim
" Specify a directory for plugins
" - For Neovim: stdpath('data') . '/plugged'
" - Avoid using standard Vim directory names like 'plugin'
call plug#begin('~/.vim/plugged')

" Make sure you use single quotes

" Shorthand notation; fetches https://github.com/junegunn/vim-easy-align
Plug 'junegunn/vim-easy-align'

" Any valid git URL is allowed
Plug 'https://github.com/junegunn/vim-github-dashboard.git'

" Multiple Plug commands can be written in a single line using | separators
Plug 'SirVer/ultisnips' | Plug 'honza/vim-snippets'

" On-demand loading
Plug 'scrooloose/nerdtree', { 'on':  'NERDTreeToggle' }
Plug 'tpope/vim-fireplace', { 'for': 'clojure' }

" Using a non-master branch
Plug 'rdnetto/YCM-Generator', { 'branch': 'stable' }

" Using a tagged release; wildcard allowed (requires git 1.9.2 or above)
Plug 'fatih/vim-go', { 'tag': '*' }

" Plugin options
Plug 'nsf/gocode', { 'tag': 'v.20150303', 'rtp': 'vim' }

" Plugin outside ~/.vim/plugged with post-update hook
Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }

" Unmanaged plugin (manually installed and updated)
Plug '~/my-prototype-plugin'

" Initialize plugin system
call plug#end()

# :PluginInstall
```


## 安装coc-nvim插件
这是一个智能补全插件
https://github.com/neoclide/coc.nvim
```bash
# 安装node
curl -sL install-node.now.sh/lts | bash
```
```bash
" Use release branch (Recommend)
Plug 'neoclide/coc.nvim', {'branch': 'release'}

" Or build from source code by use yarn: https://yarnpkg.com
Plug 'neoclide/coc.nvim', {'do': 'yarn install --frozen-lockfile'}
```

## 使用语言配置文件
```bash
# vi xx
:CocConfig
```



## 插件安装
随便进入一个文件，使用PlugInstall安装
```bash
:PluginInstall
```

## 安装Python3环境依赖
```bash
pip3 install neovim
```
这里可能会报错，报错的话，请更新pip版本和setuptools版本
https://blog.csdn.net/qq_37189082/article/details/97658103