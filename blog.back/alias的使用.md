---
title: alias的使用
categories:
  - linux
tags:
  - linux
abbrlink: b0afb621
date: 2020-03-21 14:03:02
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [alias的使用](#alias的使用)
  - [Centos中](#centos中)
    - [第一个/etc/profile文件](#第一个etcprofile文件)
    - [/etc/profile.d文件夹](#etcprofiled文件夹)
    - [bash_profile](#bash_profile)
    - [/etc/bashrc 和  ~/.bashrc](#etcbashrc-和-bashrc)
  - [mac](#mac)
  - [给自己的centos加上登录信息](#给自己的centos加上登录信息)

<!-- /code_chunk_output -->
<!-- more -->

# alias的使用

关于alias的使用，这里可以理解为某一个变量或者说某一个动作的别名(后者居多)
会涉及到几个文件夹

## Centos中
- /etc/profile
- /etc/profile.d

- ~/.bashrc
- /etc/bashrc
- ~/.bash_profile


### 第一个/etc/profile文件
profile（/etc/profile），用于设置系统级的环境变量和启动程序，在这个文件下配置会对所有用户生效。当用户登录（login）时，文件会被执行，并从/etc/profile.d目录的配置文件中查找shell设置。

```bash
# /etc/profile
  for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
      if [ -r "$i" ]; then
          if [ "${-#*i}" != "$-" ]; then·
              . "$i"
          else
              . "$i" >/dev/null
          fi
      fi
  done
```
这段代码会去/etc/profile.d/ 下面读取你的shell脚本

一般不建议在profile下面去维护自己的环境变量，建议在profile.d下面维护

```bash
# 添加变量实例
export host="www.baidu.com"
# source /etc/profile
# echo $host
# www.baidu.com
```

### /etc/profile.d文件夹
这个文件夹下面放了一些shell脚本，你也可以自己写一些shell脚本放在里面，然后source 一下就可以作为环境变量使用了，比如
```bash
#  vi /etc/profile.d/xx.sh
  name="hzj"
  alias hello="echo $name"
# source /etc/profile.d/xx.sh
# [root@node ~]# hello
# hzj
```


### bash_profile
bash_profile只有单一用户有效，文件存储位于~/.bash_profile，该文件是一个用户级的设置，可以理解为某一个用户的profile目录下。这个文件同样也可以用于配置环境变量和启动程序，但只针对单个用户有效。
和profile文件类似，bash_profile也会在用户登录（login）时生效，也可以用于设置环境变理。但与profile不同，bash_profile只会对当前用户生效。

### /etc/bashrc 和  ~/.bashrc
为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取
bashrc 文件只会对指定的 shell 类型起作用，bashrc 只会被 bash shell 调用

## mac

在mac里面/etc/profile.d/变成了 /etc/paths.d/，引用内容如下
```bash
 if [ -x /usr/libexec/path_helper ]; then
     eval `/usr/libexec/path_helper -s`
  fi
```


## 给自己的centos加上登录信息
创建一个文本，加入登录信息
```bash
#        ┏┓　　　┏┓+ +
  #　　　┏┛┻━━━┛┻┓ + +
  #　　　┃　　　　　　　┃
  #　　　┃　　　━　　　┃ ++ + + +
  #　　 ████━████ ┃+
  #　　　┃　　　　　　　┃ +
  #　　　┃　　　┻　　　┃
  #　　　┃　　　　　　　┃ + +
  #　　　┗━┓　　　┏━┛
  #　　　　　┃　　　┃
  #　　　　　┃　　　┃ + + + +
  #　　　　　┃　　　┃　　　　Codes are far away from bugs with the animal protecting
  #　　　　　┃　　　┃ + 　　　　神兽保佑,代码无bug
  #　　　　　┃　　　┃
  #　　　　　┃　　　┃　　+
  #　　　　　┃　 　　┗━━━┓ + +
  #　　　　　┃ 　　　　　　　┣┓
  #　　　　　┃ 　　　　　　　┏┛
  #　　　　　┗┓┓┏━┳┓┏┛ + + + +
  #　　　　　　┃┫┫　┃┫┫
  #　　　　　　┗┻┛　┗┻┛+ + + +
```
在/etc/profile 或者bash_profile上面添加脚本
```bash
vi /etc/profile

cat /hzj/code.text
```