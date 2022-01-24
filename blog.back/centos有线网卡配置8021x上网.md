---
title: centos有线网卡配置802.1x上网
categories:
  - linux
tags:
  - linux
abbrlink: 443b0db9
date: 2020-03-05 18:05:18
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [centos有线网卡配置802.1x](#centos有线网卡配置8021x)

<!-- /code_chunk_output -->
<!-- more -->

# centos有线网卡配置802.1x

查看版本
```bash
root:~# cat /etc/redhat-release
CentOS Linux release 7.7.1908 (Core)
```

1. 取消本机的有线活动网卡 这里举例的是ifcfg-enp3s0
```bash
sed -i -r  '/ONBOOT/s#ONBOOT.*#ONBOOT=no/g'  /etc/sysconfig/network-scripts/ifcfg-enp3s0
```
2. chkconfig --list ，查看是否有 network 的服务，如果有，执行 chkconfig --del network 删除
3. 修改 /etc/wpa_supplicant/wpa_supplicant.conf，写入如下内容：
```bash
ctrl_interface=/var/run/wpa_supplicant
ap_scan=0
network={
    key_mgmt=IEEE8021X
    eap=PEAP
    identity="YOUR_USER_NAME"
    password="YOUR_PASSWORD"
    phase2="autheap=MSCHAPV2"
}
```
4. 做检测是否成功
```bash
$ ifdown em1
$ wpa_supplicant -B -i em1 -c /etc/wpa_supplicant/wpa_supplicant.conf -D wired
$ ifup em1
$ dhclient em1
$ ip addr # 查看IP地址
$ ping baidu.com 
```
5. 设置开机启动
```bash
touch /etc/init.d/wpa_network

#!/bin/bash
# chkconfig: 2345 10 90
# description: wpa network

ifdown em1

wpa_supplicant -B -i em1 -c /etc/wpa_supplicant/wpa_supplicant.conf -D wired

ifup em1

dhclient em1
```

6. chkconfig --add wpa-network，加入到 chkconfig 中
7. chkconfig wpa-network on，开启
8. reboot 重启检测是否成功自动联网