---
title: linux周期性任务
categories:
  - linux
tags:
  - linux
abbrlink: b97ddf14
date: 2020-02-16 23:41:43
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux任务型工作](#linux任务型工作)
  - [Linux下的定时任务](#linux下的定时任务)
  - [编辑定时任务](#编辑定时任务)
    - [crontab](#crontab)
  - [crontab实例](#crontab实例)
  - [注意](#注意)

<!-- /code_chunk_output -->
<!-- more -->
# linux任务型工作

在linux长时间的工作中，为了各种各样的任务，我们把它分成两种任务形式
1. 例行性任务调度，每隔一定周期执行任务
2. 触发型或者说突发型任务调度


## Linux下的定时任务

**什么是cron**
cron是大多数linux发行版都自带的守护进程（daemon），用来重复运行某些被设定好了确定的运行时间的任务，这些任务可以是每个月运行、每周运行、每天运行，甚至是每一分钟运行。用cron执行的任务适合于24小时运行的机器，cron执行的任务会在设定好的时刻执行，当机器处于关机状态下并错过了任务执行的时间，cron任务就无法预期执行了。

**什么是crontab**
crontab(cron table的简称)既可以指cron用来定期执行特定任务所需要的列表文件，又可以指用来创建、删除、查看当前用户（或者指定用户）的crontab文件的命令。

**什么是anacron**
anacron不是守护进程，可以看做是cron守护进程的某种补充程序，anacron是独立的linux程序，被cron守护进程或者其他开机脚本启动运行，可以每天、每周、每个月周期性地执行一项任务（最小单位为天）。适合于可能经常会关机的机器，当机器重新开机anacron程序启动之后，anacron会检查anacron任务是否在合适的周期执行了，如果未执行则在anacron设定好的延迟时间之后只执行一次任务，而不管任务错过了几次周期。举个例子，比如你设定了一个每周备份文件的任务，但是你的电脑因为你外出度假而处于关机状态四周，当你回到家中开机后，anacron会在延迟一定时间之后只备份一次文件。由于发行版的不同，cron守护进程如何运行anacron会有所不同。


## 编辑定时任务
根据上面的介绍，我们可以发现cron只是一个工具，他通过被crond调用，调用定时任务并且执行这些任务，而crontab便是用来收集这些定时任务列表的工具
在Centos7中,我们可以调用crond(后台守护进程名称，d表是daemon)来控制cron
```bash
# 查看crond
systemctl | grep crond
# 开启crond
systemctl start crond.service
# 关闭crond
systemctl stop cornd.service
# 重启crond
systemctl restart crond.service
```

### crontab
开启cron定时任务之后，我们需要向定时列表中添加任务，这时候我们就要使用到我们的crontab
```bash
# 查看现有的任务
crontab -l
```
添加定时任务
```bash
# 进入任务列表
crontab -e
# 先写好.cron文件使用crontab 加入
crontab jobs.cron
# 删除所有定时任务
crontab -r
# 查看作业日志
ls /var/log	| grep cron



# 任务编写格式
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```
编写的格式基本如上所示，但是在不同的cron应用存在很大的差异，比如js中的cron模块

|*   | 任意时间点都执行  |
| ------------ | ------------ |
| ？  | 不指定，任意。仅用于 日(月)和日(周)。0 0 5 \* ? 代表每个月的第5天零点，不论星期几。0 0 ? \* 1 代表每周一，不论是当月的哪天。  |<font style="color:red">为啥还要？</font>
| ,  | 多个值的分隔符，比如  ? 1,5,10 \* \* \* 表示每天的1点5点和10点  |
| -  | 代表连续值 比如 ? 1-20 \* \* \* 表示每天的1-20点  |
| / |  代表步长 比如 ？1/2 \* \* \* 表示每天从1点开始过两个小时执行1次  \*/1 \* \* \* \* 每分钟执行一次  |
| L  |  最后一天。可以是每月的最后一天或者每周的最后一天。如果用在 天(周)字段，并且前面加数字，则表示最后一个周N。例如5L，表示最后一个周五（5表示周五，L表示最后）。  |
| W  |  工作日，指周一到周五的任意一天  |
| #  |  表示第几个的意思，例如 6#3，表示当月第3个星期六（6表示周六，3表示第3个）  |


## crontab实例
```bash
* * * * * echo 'xxx' > xx.log 每分钟执行后一次
*/1 * * * * echo 'xxx' > xxx.log 每分钟执行一次
30 * * * * echo 'xxx' > xxx.log 每30分钟执行一次
0/5 * * * *	 每5分钟执行一次，且仅在0,5,15,20...55分执行
5 0 * * *	每天的00:05执行一次
```



## 注意
添加定时任务时，最好不要在crontab -e下编辑任务列表，因为crontab -e只会记录当前登录用户的定时列表
更好的解决办法是： 由于cron每次执行都会查询一边/var/spool/cron下的文件或者/etc/cron.d/下的cron文件，因此可以在/etc/cron.d/下创建一个用于记录定时列表的文件，然后重启crond服务