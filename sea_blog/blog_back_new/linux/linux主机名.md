---
title: linux主机名
date: 2020-12-09 11:26:42
categories: [linux,]  
tags: [linux,linux系统]
---


<!-- more -->
# linux主机名

三种类型的主机名
- 静态主机名也称为内核主机名，是系统在启动时从/etc/hostname内自动初始化的主机名。
- 瞬态主机名是在系统运行时临时分配的主机名。
- 灵活主机名则允许使用特殊字符的主机名。

## 命令

```bash
# hostnamectl -h

  -h --help              显示帮助
     --version           显示安装包的版本
     --transient         修改临时主机名
     --static            修改瞬态主机名
     --pretty            修改灵活主机名
  -P --privileged        在执行之前获得的特权
     --no-ask-password   输入密码不提示
  -H --host=[USER@]HOST  操作远程主机

Commands:
  status                 显示当前主机名设置
  set-hostname NAME      设置系统主机名
  set-icon-name NAME     为主机设置icon名
  set-chassis NAME       设置主机平台类型名
```

查看状态
```bash
hostnamectl status
```

修改静态主机名
```bash
hostnamectl  --static set-hostname Linuxprobe
```