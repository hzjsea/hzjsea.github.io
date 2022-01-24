---
title: linux命令ncat
date: 2020-12-11 10:56:41
categories: [linux,]  
tags: [linux,linuxcc]
---


<!-- more -->

# linux命令ncat

https://cloud.tencent.com/developer/article/1432599
https://www.cnblogs.com/chengd/p/7565280.html
https://www.sqlsec.com/2019/10/nc.html


## ncat加密传输文件
```bash
# 服务端，使用 mcrypt 工具加密数据，需要输入密码
nc localhost 1567 | mcrypt –flush –bare -F -q -d -m ecb > file.txt

# 客户端，使用 mcrypt 工具解密数据，需要输入密码
mcrypt –flush –bare -F -q -m ecb < file.txt | nc -l 1567
```

## ncat 传输文件
```bash
# 接收端
nc -l -p 8080 > test.txt
# 发送端
nc 192.168.1.8 8080 < test.txt
```