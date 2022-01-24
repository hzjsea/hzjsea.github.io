---
title: linux命令firewalld
date: 2020-11-30 11:35:40
categories: [linux,]  
tags: [linuxcc,linux]
---


<!-- more -->

# firewalld防火墙
centos7后开始引入firewalld防火墙,和iptables一样都是防火墙管理工具,真正使用到的时候centos的netfilter网络过滤器, firewalld防火墙在功能和模式上更加详细一点.


## 使用
```bash
ervice firewalld restart # 重启
service firewalld start # 开启
service firewalld stop # 关闭
```



https://wangchujiang.com/linux-command/c/firewall-cmd.html
https://www.jianshu.com/p/70f7efe3a227
https://www.cnblogs.com/kerrycode/p/12392984.html

## 错误

### firewalld无法开启
```bash
Failed to start firewalld.service: Unit not found.
# 这个原因是因为没有安装firewalld
```
解决方法
```bash
yum install firewalld firewall-config
# 关闭iptables
systemctl status iptables.service
systemctl stop iptables.service
# 查看firewalld状态
systemctl status firewalld
```

### firewalld状态错误
查看systemctl status 状态如下
```bash
[root@ops ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Mon 2020-11-30 11:48:38 CST; 2s ago
     Docs: man:firewalld(1)
  Process: 8119 ExecStart=/usr/sbin/firewalld --nofork --nopid $FIREWALLD_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 8119 (code=exited, status=0/SUCCESS)

Nov 30 11:48:38 ops.novalocal systemd[1]: Starting firewalld - dynamic firewall daemon...
Nov 30 11:48:38 ops.novalocal firewalld[8119]: ERROR: Exception DBusException: org.freedesktop.DBus.Error.AccessDenied: Connection ":1.374" is not allowed to own the service "org.fedoraproject.FirewallD1" due to security policies in the configuration file
Nov 30 11:48:38 ops.novalocal systemd[1]: Started firewalld - dynamic firewall daemon.
Hint: Some lines were ellipsized, use -l to show in full.
```
```
ERROR: Exception DBusException: org.freedesktop.DBus.Error.AccessDenied: Connection ":1.374" is not allowed to own the service "org.fedoraproject.FirewallD1" due to security policies in the configuration file
```
重启dbus
```bash
systemctl restart dbus
systemctl restart firewalld
```