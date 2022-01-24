---
title: docker-compose
categories:
  - docker
tags:
  - docker
abbrlink: f9b84192
date: 2020-02-07 14:18:36
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker-compose](#docker-compose)
  - [三大概念](#三大概念)
  - [docker-compose安装](#docker-compose安装)
  - [使用docker-compose搭建wordpress](#使用docker-compose搭建wordpress)
  - [docker-compose命令](#docker-compose命令)

<!-- /code_chunk_output -->
<!-- more -->

# docker-compose
- docker-compose是一个工具
- 通过yml文件定义多容器的docker应用
- 通过一条命令就可以根据yml文件的定义去创建或者管理多个容器


## 三大概念
- services


一个service代表了一个container 这个container可以从dockerhub的image创建，或者从本地的dockerfile build出来的image创建
```bash
# compose部分
services:
  db:
    images: mysql:8.0
    volumes:
      - "db-data:/var/lib/mysql"
    networks:
      - back-tier
# 类似于COMMAND命令
docker run -d --network back-tier -v db-data:/var/lib/mysql mysql:8.0


# compose部分
services:
  worker:
    build: ./worker
    links:
      - db 
      - redis
    network:
      - back-tier
```
- networks
- volumes


## docker-compose安装
https://docs.docker.com/compose/install/
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose 
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# 查看版本
docker-compose --version
```

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

## docker-compose命令
```bash 
# 根据docker-compose.yml文件运行
docker-compose -f docker-compose.yml up
# 后台运行
docker-compose up -d
# 查看
root:~# docker-compose ps
      Name                    Command               State    Ports
------------------------------------------------------------------
root_db_1          docker-entrypoint.sh --def ...   Exit 0
root_wordpress_1   docker-entrypoint.sh apach ...   Exit 0

# 停止
docker-compose stop
# 开启
docker-compose start
# 停止并删除
docker-compose down
# 查看compose的容器
docker-compose images
# 进入compose的容器
docekr-compose exec IMAGE COMMAND

```