---
title: docker搭建wordpress
categories:
  - docker
tags:
  - docker
abbrlink: a8b92e4
date: 2020-02-07 13:26:31
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker搭建wordpress](#docker搭建wordpress)
  - [使用docker-compose搭建wordpress](#使用docker-compose搭建wordpress)

<!-- /code_chunk_output -->
<!-- more -->

# docker搭建wordpress

1. 安装mysql、wordpress镜像
```bash
docker pull mysql
docker pyll wordpress 
```
2. 创建mysql
需要做一些初始化的配置，建议在dockerhub上查看
https://hub.docker.com/_/mysql
```bash
docker run -it -d --name mysql \
-v db_data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=wordpress \
mysql:8.0  
```
3. 创建wordpress
需要连接数据库与暴露端口转换
```bash
docker run -d -e WORDPRESS_DB_HOST=mysql:3306 \
--name wordpress \
--link mysql  \
-e WORDPRESS_DB_USER=root \
-e WORDPRESS_DB_PASSWD=root \
-p 8888:80 \
wordpress 
```
<font color='red'>未成功，一直提醒数据库错误，不知道为什么? 是因为mysql没有加--default-auth=mysql_native_password这个嘛</font>

## 使用docker-compose搭建wordpress
```bash
version: "3"
services:

   db:
     image: mysql:8.0
     command:
      - --default_authentication_plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci     
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data: 
```