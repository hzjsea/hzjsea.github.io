---
title: nginx编译安装
categories:
  - nginx
tags:
  - nginx
abbrlink: 14b81227
date: 2020-05-12 13:53:31
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [nginx编译安装](#nginx编译安装)

<!-- /code_chunk_output -->

<!-- more -->

# nginx编译安装

```bash
# 安装nginx编译所需要的库
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
yum -y install pcre pcre-devel
# 进入编译目录
cd /usr/local/src
# 下载稳定版本的nginx
wget http://nginx.org/download/nginx-1.16.1.tar.gz
# 解压nginx
tar -zxvf nginx-1.16.1.tar.gz && cd  nginx-1.16.1
# 运行配置文件，将Nginx安装到/usr/local/nginx下面
./configure --prefix=/usr/local/nginx
# 编译
make && make install
# 软连接
ln -s /usr/local/nginx/sbin/nginx /usr/bin
# 启动
nginx
# 设置开机启动
echo "/usr/local/nginx/sbin/nginx" >> /etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local

# 查看当前运行的nginx的安装目录
whereis nginx

# 之前没有了解好nginx的安装路径以及项目结构，安装了很多遍nginx 后来用whereis找的时候，发现只能找到当前运行的nginx的安装路径
```