---
title: neovim配置
categories:
  - vim
tags:
  - vim
abbrlink: 8673b365
date: 2020-02-25 22:16:30
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [neovim配置](#neovim配置)
  - [在centos服务器上搭建自己的neovim环境](#在centos服务器上搭建自己的neovim环境)
  - [错误](#错误)
    - [vim version 8](#vim-version-8)
  - [你可能需要的一些网址](#你可能需要的一些网址)

<!-- /code_chunk_output -->
<!-- more -->

# neovim配置

## 在centos服务器上搭建自己的neovim环境

```bash
yum upgrade
yum install python3 git  
curl -LO https://github.com/neovim/neovim/releases/download/stable/nvim.appimage
## 添加权限
chmod 777 ./nvim.appimage
./nvim.appimage --appimage-extract
ln -s ./squashfs-root/usr/bin/nvim /usr/bin/nvim 


# 下载配置
mkdir ~/.config
git clone git://github.com/rafi/vim-config.git ~/.config/nvim
cd ~/.config/nvim
sh venvs.sh
make test
make

# 安装deoplete-tabnine
cd ~/.config/nvim
touch config/local.plugin.yaml

# cat > ./local.plugin.yaml << EOF
> ---
> - repo: tbodt/deoplete-tabnine
>   build: ./install.sh
> EOF

make update 
```


## 错误

### vim version 8 
动手编译安装vim 8+
```bash
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

## 你可能需要的一些网址
如何安装neovim
https://github.com/neovim/neovim/wiki/Installing-Neovim
vim 配置文件
https://github.com/PegasusWang/vim-config