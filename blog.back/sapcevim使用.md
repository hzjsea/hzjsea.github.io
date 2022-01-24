---
title: spacevim使用
categories:
  - vim
tags:
  - vim
abbrlink: 572cc941
date: 2020-05-20 05:07:31
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->


<!-- more -->


# spacevim


## 加快提示菜单栏
spacevim提供了一个提示的菜单栏，默认延时是1000ms,用户可以根据自己的需求来改变延时
延时的意思就是要等待一定时间后才会显示，官方的用意是来给用户忘记指令的时候，再去看，一开始初学的话，可以调低一点，因为使用评率蛮高的 

首先打开一个文件
```bash
:set timeoutlen  # 查看当前延时的时间
:set timeoutlen=10 # 设置延时时间为10ms
```