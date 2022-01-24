---
title: linux操作hostname.md
date: 2020-09-23 10:53:29
categories: [linux,]  
tags: [linux,linuxcommand]
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->

# linux操作hostname
在操作linux系统的过程中，难免要修改和查看hostname，这里介绍一下hostname的相关操作那个

## 临时修改主机名
```bash
sudo hostname 主机名 # 重启后还原
# 如果你是ssh 上去的 需要退出重进才会更改
```

## 永久修改主机名
```bash
echo helloworld > /etc/hostname
echo 127.0.0.1 helloworld > /etc/hosts
```

## 永久修改主机名
```bash
sudo hostnamectl set-hostname helloworld
echo 127.0.0.1 helloworld > /etc/hosts
```