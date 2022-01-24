---
title: mysql in docker
date: 2021-04-02 17:15:58
categories:  [docker,mysql]
tags: [docker,mysql]
---


<!--more-->


# mysql in docker

分析`mysql in docker` 启动的脚本,也就是官方的mysql dockerfile
https://www.cnblogs.com/ivictor/p/4832832.html


docker run -p 12345:3306 --name mysql2 -v /conf/mysql/conf:/etc/mysql/conf.d -v /conf/mysql/logs:/logs -v /conf/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
