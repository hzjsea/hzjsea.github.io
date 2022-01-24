---
title: 常见错误
categories:
  - linux
tags:
  - linux
abbrlink: c10f304f
date: 2019-12-29 13:13:35
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [centos7错误总结](#centos7错误总结)
  - [yum不支持python3](#yum不支持python3)
  - [可以通外网，但是无法ping百度](#可以通外网但是无法ping百度)
  - [linux中如何恢复到后台进程](#linux中如何恢复到后台进程)
  - [ssh配置的问题](#ssh配置的问题)

<!-- /code_chunk_output -->

<!-- more -->

# centos7错误总结

## yum不支持python3
```bash 
[root@gogogo alpaca]# yum
  File "/bin/yum", line 30
    except KeyboardInterrupt, e:
                            ^
SyntaxError: invalid syntax
  File "/usr/libexec/urlgrabber-ext-down", line 28
    except OSError, e:
```
**解决办法**
```bash 
sed -i 's:bin/python:bin/python2.7:' /usr/bin/yum
sed -i 's:bin/python:bin/python2.7:' /usr/libexec/urlgrabber-ext-down
```

## 可以通外网，但是无法ping百度
包括无法使用yum wget 等下载工具 
或者报错 cannot find a valid baseurl for repo:base/7/x86_64 的解决方法

**解决办法**
原因 - > DNS的问题

```bash
echo nameserver 114.114.114.114 > /etc/reslov.conf
```

## linux中如何恢复到后台进程
linux 下我们如果想一个任务或者程序进行调度
bg 将一个后台暂停的命令，变成继续执行
fg 将后台的命令调至前台继续运行
jobs 查看当前在后台运行的命令
c-z 将前台命令放到后台运行 并且暂停
nohup command & 在后台不断运行command命令

## ssh配置的问题
<font color='blue'>这个问题主要是存在于交换机ssh登陆交换机的过程</font>
ssh配置过程中，配置了密码登陆，但是出现下面的错误
```bash
<ONI-KC-R01-CR1>ssh 10.0.0.166
The server's host key does not match the local cached key. Either the server administrator has changed the host key, or you connected to another server pretending to be this server. Please remove the local cached key, before logging in!
```

主要问题是一开始用ssh登陆交换机的时候，选择了保存key
```bash 
<ONI-KC-R01-CR1>ssh 10.0.0.166
Username: tt
Press CTRL+C to abort.
Connecting to 10.0.0.166 port 22.
The server is not authenticated. Continue? [Y/N]:y
Do you want to save the server public key? [Y/N]:y
```
这样导致的问题就是在当前交换机上会保存一个登陆另一个交换机的key
```bash
#
public-key peer 10.0.0.166
 public-key-code begin
   30819F300D06092A864886F70D010101050003818D0030818902818100DF3E1240D158E197
   CE0712173C8591883F88CC925B1C0CFC63C779E9F531C3B7E409BBB2CCED954C09A339BE78
   46B1497E4771EDD7E88D2E380C50F37A6EC9254E6B27EC7AFFE6DCDFC1EA230D16C4DC9BB8
   B831A2F0CB578B736566052E830A582836AA9BFFE0821CE7CB43F74077D38B20437479F260
   A5D0550CA7D3D0747F020001
 public-key-code end
 peer-public-key end
# 
```
于是当你下次选择使用ssh+密码登陆交换机的时候就会发现错误

<font color='red'>解决</font>
删除当前交换机的key  并在下次登陆的时候选择
```bash
<ONI-KC-R01-CR1>ssh 10.0.0.166
Username: tt
Press CTRL+C to abort.
Connecting to 10.0.0.166 port 22.
The server is not authenticated. Continue? [Y/N]:y
Do you want to save the server public key? [Y/N]:n
```

