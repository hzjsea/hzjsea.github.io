---
title: linux系统时间
date: 2020-12-08 15:47:00
categories: [linux,]  
tags: [linux系统,linux]
---


<!-- more -->

# linux系统时间

> 时区

整个地球被分为二十四时区，每个时区都有自己的本地时间。为了克服时间上的混乱，1884年在华盛顿召开的一次国际经度会议（又称国际子午线会议）上，规定将全球划分为24个时区（东、西各12个时区）。使用一个统一的时间， 称为通用协调时(UTC, Universal Time Coordinated)。UTC与格林尼治平均时(GMT, Greenwich Mean Time)一样，都与英国伦敦的本地时相同
所以东八区(北京)就表示为`UTC+8标识` 西八区 `UTC-8`

> CST


所谓的CST时间代表四个不同的时区：

Central Standard Time (USA) UT-6:00 美国标准时间
Central Standard Time (Australia) UT+9:30 澳大利亚标准时间
China Standard Time UT+8:00 中国标准时间
Cuba Standard Time UT-4:00 古巴标准时间


> 系统时钟与硬件时钟的区别

Linux时钟分为系统时钟(system clock)和硬件(real time clock 也就是RTC)时钟。系统时钟是指当前Linux Kernel中的时钟，而硬件时钟则是主板上由电池供电的时钟，这个硬件时钟可以在BIOS中进行设置。当Linux启动时，硬件时钟会去读取系统时钟的设置，然后系统时钟就会独立于硬件运作。
在计算机通电运行的时候,是不会去跑电池里面的电的, 当计算机不持续通电的时候,电池就会发生作用,BIOS电池一般可以使用几年,如果BIOS中的数据就会恢复出厂设置,轻量数据,也就是一些BIOS启动项等等, 不影响机器内部数据的

> 硬件时间如何同步系统时间

首先硬件时间是记录在电池上的时钟,开启启动后硬件时间会读取系统时间 然后实现同步,在不重启的情况下,只能使用命令行进行同步


> 系统时钟与硬件时钟对应的指令

```bash
# 查看系统时间
date
# 设置系统时间
date --set "07/07/06 10:19" #(月/日/年时:分:秒)

# 查看硬件时间
clock --show # hwclock --show
# 设置硬件时间
hwclock --set --date="07/07/06 10:19" 
clock --set --date="07/07/06 10:19"

# 硬件时间与系统时间同步
hwclock --hctosys # (hc 表示硬件时间 sys表示系统时间)
# 系统时间同步到硬件时间上
hwclock --systohc

```

## linux中关于时区的文件

```bash
/usr/share/zoneinfo/：此目录包含时区文件。
/etc/localtime：此文件是时区文件的符号链接。
/etc/timezone：该文件在基于Debian的系统上保存时区名称。
/etc/sysconfig/clock：此文件在基于RHEL的系统上保存时区名称。
```

> /usr/share/zoneinfo/

![2020-12-08-16-19-03](http://noback.upyun.com/2020-12-08-16-19-03.png)

> /etc/localtime

![2020-12-08-16-20-13](http://noback.upyun.com/2020-12-08-16-20-13.png)


## date命令

date命令是显示当前时间
```bash
>> date
Tue Dec  8 03:22:53 EST 2020
# 其中EST表示的是一个时区的简称
```

查看机器本地的时区
```bash
>>  date -R
Tue, 08 Dec 2020 03:30:28 -0500
```
这里就表示的是西五区

查看UTC的时间

```bash
>> date -u
Mon Dec  9 07:00:54 UTC 2019
```

## timedatectl命令

查看现有时区

```bash
>> timedatectl 

      Local time: Mon 2019-12-09 15:10:52 CST
  Universal time: Mon 2019-12-09 07:10:52 UTC
        RTC time: Mon 2019-12-09 15:10:52
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: yes
      DST active: n/a
```

列出所有可以设置的时区

```bash
>> timedatectl list-timezones
```

设置本地时区
```bash
>> date
Wed Dec  9 10:31:28 CST 2020
>> timedatectl set-timezone Europe/London
# 修改的是本地的时区
>> date
Wed Dec  9 02:31:46 GMT 2020
```

> 将时间同步到远程的NTP服务器
```bash
timedatectl set-ntp true
```

> 禁用NTP时间同步
```bash
timedatectl set-ntp false
```



## NTP命令
NTP(Network Time Protocol,网络时间协议) 是用来使网络中的各个计算机时间同步的一种协议。它的用途是把计算机的时钟同步到世界协调时UTC，其精度在局域网内可达0.1ms，在互联网上绝大多数的地方其精度可以达到1-50ms。(1s=1000ms) NTP服务器就是利用NTP协议提供时间同步服务的.

> 为了避免主机时间因为长期运行下所导致的时间偏差，进行时间同步（synchronize）的工作是非常必要的

linux服务器既可以作为NTP时间服务器,也可以作为NTP时间客户端 , ntp是第三方服务,需要使用yum安装
```bash
yum install ntpdate -y
```

> 同步远程ntp服务器

```bash
ntp 192.168.20.1
ntp time.ntp.org
```
但这样的同步，只是强制性的将系统时间设置为ntp服务器时间。如果cpu tick有问题，只是治标不治本。所以，一般配合cron命令，来进行定期同步设置。比如，在crontab中添加：
每天十二点同步一次时间
```bash
0 12 * * * * /usr/sbin/ntpdate192.168.0.168.1
```

> 搭建时钟服务器

安装软件
```bash
yum install -y ntp ntpdate
```
配置NTP服务
ntp配置文件的路径是/etc/ntp.conf
```bash
>>  whereis ntp
ntp: /etc/ntp /etc/ntp.conf

>> cp /etc/ntp.conf /etc/ntp.conf.bak
>> vim /etc/ntp.conf
restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap  # 这个表示该网段内的服务器都可以使用该ntp服务器同步时间.
server 192.168.1.2 # 同时添加指定一台内网服务器作为本地时钟服务器用于无法链接外网时，其它服务器同步时间是以该服务器为准
# 启动ntpd服务
systemctl start ntpd
```

查看ntpd服务状态
```bash
>> ntpstat
synchronised to NTP server (119.28.206.193) at stratum 3
   time correct to within 37 ms
   polling server every 1024 s
```


## 常用操作

> 校准上海时间
```bash 
# 直接强制覆盖, 这样就不用多做一步了(删除原来存在的时区)
ln -snf  /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
```

> docker同步主机时间

```bash
# command
docker run -p 3306:3306 --name mysql -v /etc/localtime:/etc/localtime

# dockerfile
# 方法1 # 添加时区环境变量，亚洲，上海 
ENV TimeZone=Asia/Shanghai 
# 使用软连接，并且将时区配置覆盖/etc/timezone 
RUN ln -snf /usr/share/zoneinfo/$TimeZone /etc/localtime && echo $TimeZone > /etc/timezone 
# 方法2 
# CentOS 
RUN echo "Asia/shanghai" > /etc/timezone 
# Ubuntu 
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# docker-compose
# 方法1
environment: 
    TZ: Asia/Shanghai

# 方法2
volumes:
    - /etc/timezone:/etc/timezone
    - /etc/localtime:/etc/localtime
```
