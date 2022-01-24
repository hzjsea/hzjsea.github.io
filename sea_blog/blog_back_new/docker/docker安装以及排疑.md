---
title: docker安装
date: 2020-11-30 14:47:04
categories: [docker,]  
tags: [docker,]
---


<!-- more -->


## 安装
<font color='red'>这个方法已经不实用了</font>

```bash

# centos7老版本安装
yum install docker -y 
# 查看版本
docker --version
Docker version 19.03.1, build 74b1e89
```

## 安装docker-ce 社区版本
docker 具有社区版和企业版，在企业版中会提供一起额外的收费服务
https://docs.docker.com/install/linux/docker-ce/centos/
```bash
# centos7安装docker-ce
# 卸载旧版本docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 设置存储库
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2 -y 
# 添加下载源
# 国外源
sudo yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo -y 
# 阿里源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -y 
# 安装docker社区版本
sudo yum install docker-ce docker-ce-cli containerd.io -y
# 启动docker
sudo systemctl start docker
# 测试
docker run hello-world
# 查看版本
docker version
# 启动docker服务
systemctl start docker
```

## 错误


### docker无法ping通外网
网关也ping不通
```bash
[root@fileserver ~]# ping 172.17.78.2
PING 172.17.78.2 (172.17.78.2) 56(84) bytes of data.
From 172.17.78.1 icmp_seq=1 Destination Host Unreachable
From 172.17.78.1 icmp_seq=2 Destination Host Unreachable
From 172.17.78.1 icmp_seq=3 Destination Host Unreachable
From 172.17.78.1 icmp_seq=4 Destination Host Unreachable
```
解决方法如下


```bash

## 方法0,openstack中的机器就是这么解决的,重启大法
reboot

## 方法1
# 关闭防火墙
sudo systemctl stop iptables
sudo systemctl disable iptables
sudo systemctl stop firewalld
sudo systemctl disable firewalld
# 关闭selinux
vi /etc/selinux/config


## 方法2
## 开启主机网络,不建议,最好做隔离
docker run -d --name test --net=host image:1.0

## 方法3
## 升级系统内核

# uname -r
3.10.0-327.el7.x86_64
## 导入内核源
rpm -import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install -y kernel-ml

## 安装完后, 查看内核列表
# cat /boot/grub2/grub.cfg |grep menuentry
menuentry 'CentOS Linux (5.7.8-1.el7.elrepo.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-327.el7.x86_64-advanced-3bfef048-3886-4956-9ec3-7bdbfa7f6726' {
menuentry 'CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-327.el7.x86_64-advanced-3bfef048-3886-4956-9ec3-7bdbfa7f6726' {
...

## 设置默认启动高版本内核, 注意替换版本为上面查询得到的版本号
grub2-set-default 'CentOS Linux (5.7.8-1.el7.elrepo.x86_64) 7 (Core)'

## 重启
reboot

# uname -r
5.7.8-1.el7.elrepo.x86_64

```