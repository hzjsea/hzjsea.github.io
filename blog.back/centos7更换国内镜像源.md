---
title: centos7更换国内镜像源
categories:
  - linux 
tags:
  - linux
abbrlink: 90eb98f2
date: 2020-03-08 22:34:01
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [centos7更换国内镜像源](#centos7更换国内镜像源)
  - [步骤](#步骤)
    - [备份之前的镜像源](#备份之前的镜像源)
    - [下载源](#下载源)
    - [清除缓存](#清除缓存)
    - [生成新的缓存](#生成新的缓存)
    - [安装epel源](#安装epel源)

<!-- /code_chunk_output -->
<!-- more -->

# centos7更换国内镜像源

## 步骤

### 备份之前的镜像源
```bash 
cd /etc/yum.repos.d/
mkdir repo_bak
mv *.repo repo_bak/
```

### 下载源
```bash
# 阿里云
wget http://mirrors.aliyun.com/repo/Centos-7.repo
# 网易云
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
```


### 清除缓存
```bash
yum clean all
```

### 生成新的缓存
```bash 
yum makecache
```

### 安装epel源
```bash
 yum list | grep epel-release
 yum install -y epel-release
```