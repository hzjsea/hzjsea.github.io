---
title: linux命令整理
categories:
  - linux
tags:
  - linux
abbrlink: 9487367a
date: 2020-03-21 12:31:19
---



<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux常用命令整理](#linux常用命令整理)
  - [系统](#系统)
    - [查看linux版本信息](#查看linux版本信息)
    - [查看系统内存](#查看系统内存)
    - [chroot](#chroot)
    - [dmesg](#dmesg)
    - [lspci](#lspci)
    - [lsusb](#lsusb)
    - [lsblk](#lsblk)
    - [blkid](#blkid)
    - [lscpu](#lscpu)
    - [lsmod](#lsmod)
  - [网络](#网络)
    - [netstat命令可以帮助查找本机的网络状态](#netstat命令可以帮助查找本机的网络状态)
    - [traceroute 路由追踪](#traceroute-路由追踪)
      - [实战](#实战)
      - [工作原理](#工作原理)
    - [mtr 路由追踪](#mtr-路由追踪)
      - [其他用法](#其他用法)
    - [域名查询工具 dig](#域名查询工具-dig)
  - [监听](#监听)
    - [lsof可以查看系统打开的文件](#lsof可以查看系统打开的文件)
    - [ps 查找与进程相关的PID号](#ps-查找与进程相关的pid号)
  - [文件](#文件)
    - [mkdir 创建目录](#mkdir-创建目录)
    - [wget下载文件工具](#wget下载文件工具)
    - [tar解压缩命令](#tar解压缩命令)
    - [find 查找文件](#find-查找文件)
    - [cp 复制文件](#cp-复制文件)
    - [ldd 查看动态库依赖关系](#ldd-查看动态库依赖关系)
    - [alias 查看快捷输入](#alias-查看快捷输入)
    - [ls 查看当前目录文件](#ls-查看当前目录文件)
    - [du查看目录或文件所占用磁盘大小](#du查看目录或文件所占用磁盘大小)
    - [文件传输  scp命令](#文件传输-scp命令)
  - [命令](#命令)
    - [xargs命令](#xargs命令)

<!-- /code_chunk_output -->
<!-- more -->


# linux常用命令整理


## 系统
### 查看linux版本信息
1.使用lsb_release 查看
```sh
lsb_release -a
# 出现错误 bash: lsb_release: command not found...
# 安装lsb_release
yum install -y readhat-lsb
lsb_release -a
```
2.uname 适合所有的linux发行版本
```sh
[root@CodeSheep ~]# uname
Linux
[root@CodeSheep ~]# uname -r
3.10.0-957.5.1.el7.x86_64
[root@CodeSheep ~]# uname -a
Linux CodeSheep 3.10.0-957.5.1.el7.x86_64 #1 SMP Fri Feb 1 14:54:57 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
[root@CodeSheep ~]# 
```

3.使用redhat-release
```sh
[root@CodeSheep ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
```

4.查看Cetnos版本于redhat对应版本的命令
```sh
[root@CodeSheep ~]# cat /proc/version 
Linux version 3.10.0-957.5.1.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC) ) #1 SMP Fri Feb 1 14:54:57 UTC 2019
```

### 查看系统内存
```bash
cat /proc/meminfo | head -3
```
### chroot
chroot主要用于修改系统的根分区，linux系统中常常是以/为根目录的

功能:
- 在忘记root密码的时候，进入单用户修改密码
- 隔离环境
在经过 chroot 之后，在新根下将访问不到旧系统的根目录结构和文件，这样就增强了系统的安全性。这个一般是在登录 (login) 前使用 chroot，以此达到用户不能访问一些特定的文件
系统读取的是新根下的目录和文件，这是一个与原系统根下文件不相关的目录结构。在这个新的环境中，可以用来测试软件的静态编译以及一些与系统不相关的独立开发。
- 构建临时系统
lfs系统构建中，为了隔离原系统的目录，会用到chroot

用chroot构建linux临时环境
```bash
# 复制依赖库
cd /root/xx
mkdir /root/xx/bin
cp -av /bin/bash /root/xx/bin/bash
mkdir /root/xx/lib64/
# 复制依赖库文件
cp -av /lib64/ld-linux-x86-64.so.2 /lib64/ld-2.12.so \
> /lib64/libc.so.6 /lib64/libc-2.12.so /lib64/libdl.so.2  \
> /lib64/libdl-2.12.so /lib64/libtinfo.so.5 \
> /lib64/libtinfo.so.5.7 /mnt/123/lib64 \
>  /mnt/123/lib64
……

chroot /root/xx
```
https://www.ibm.com/developerworks/cn/linux/l-cn-chroot/index.html
https://www.cnblogs.com/sparkdev/p/8556075.html


### dmesg
dmesg 命令可以列出系统启动时硬件检测和驱动加载的情况，以及后续由内核生成的其他信息。

```bash
[root@gogogo ~]# dmesg
[    0.000000] Linux version 5.3.0-1.el7.elrepo.x86_64 (mockbuild@Build64R7) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)) #1 SMP Mon Sep 16 08:44:46 EDT 2019
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-5.3.0-1.el7.elrepo.x86_64 root=UUID=f13d84b4-c756-4d89-9d5e-6b534397aa14 ro console=tty0 crashkernel=auto console=ttyS0,115200
[    0.000000] Disabled fast string operations
[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
[    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
[    0.000000] x86/fpu: Enabled xstate features 0x7, context size is 832 bytes, using 'standard' format.
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009bbff] usable
[    0.000000] BIOS-e820: [mem 0x000000000009bc00-0x000000000009ffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x000000007fffcfff] usable
[    0.000000] BIOS-e820: [mem 0x000000007fffd000-0x000000007fffffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000fffbc000-0x00000000ffffffff] reserved
[    0.000000] NX (Execute Disable) protection: active
[    0.000000] SMBIOS 2.4 present.
[    0.000000] DMI: RDO Project OpenStack Nova, BIOS 0.5.1 01/01/2007
[    0.000000] Hypervisor detected: KVM
[    0.000000] kvm-clock: Using msrs 4b564d01 and 4b564d00

```

**内核加载开始记录，记录时打印出来**
我们可以使用他来看是否有识别到一些外部加入的设备，比如识别USB 鼠标 串口等等
[1206367.860755] usb 2-1.6: FTDI USB Serial Device converter now attached to ttyUSB0  比如这一条就是识别到了一个USB设备,检测设备是否可用,echo "test" >>  /dev/ttyUSB0


**常用命令**
因为dmesg是从内核加载开始后记录的，记录的内容太长，如果我们要过滤 ，还要加入管道和grep来筛选

```shell
dmesg | head -20 # 只输出前20行日志
dmesg | tail -20 # 只输出后20行日志
dmesg | grep -i usb # 只筛选出有usb日志行的 -i 忽略大小写
dmesg | grep -E -i "eth|sda" # 只筛选包含字符串“eth”或“sda”的日志行
```

### lspci
lspci 命令可以列出系统中的 PCI 总线及其连接的设备。该命令的输出如下：
<font color='red'>什么是PCI</font>
https://zh.wikipedia.org/wiki/%E5%A4%96%E8%AE%BE%E7%BB%84%E4%BB%B6%E4%BA%92%E8%BF%9E%E6%A0%87%E5%87%86

```bash
[root@gogogo ~]# lspci
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
00:01.1 IDE interface: Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
00:01.2 USB controller: Intel Corporation 82371SB PIIX3 USB [Natoma/Triton II] (rev 01)
00:01.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 03)
00:02.0 VGA compatible controller: Cirrus Logic GD 5446
00:03.0 Ethernet controller: Red Hat, Inc. Virtio network device
00:04.0 SCSI storage controller: Red Hat, Inc. Virtio block device
00:05.0 SCSI storage controller: Red Hat, Inc. Virtio block device
00:06.0 RAM memory: Red Hat, Inc. Virtio memory balloon
```
输出信息中包括声音（Multimedia audio controller），USB设备（USB controller），视频输出（VGA compatible controller），有线网卡（Ethernet controller）和磁盘（SATA controller）等。
如需获取更详尽的信息，还可以在该命令后增加一至多个 -v 选项（如 -vvv）。

### lsusb
查看usb设备的情况  

找出连接了多少usb设备
```shell
find /dev/bus
```

列出某台USB设备的详细信息
```shell
lsusb -D /dev/bus/usb/001/002
```
以树层结构列出 USB 设备
```shell
lsusb -t 
```

如需获取更详尽的信息，还可以在该命令后增加一至多个 -v 选项（如 -vvv）

### lsblk
lsblk 命令用于列出所有可用块设备的信息，还能显示他们之间的依赖关系。块设备有硬盘，闪存盘，cd-ROM等等。
```bash
[root@test-ceph ~]# lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda               8:0    0 447.1G  0 disk
├─sda1            8:1    0   200M  0 part /boot/efi
├─sda2            8:2    0     1G  0 part /boot
└─sda3            8:3    0   446G  0 part
  ├─centos-root 253:0    0    50G  0 lvm  /
  ├─centos-swap 253:1    0   7.8G  0 lvm  [SWAP]
  └─centos-home 253:2    0 388.1G  0 lvm  /home
sdb               8:16   0   1.8T  0 disk
├─sdb1            8:17   0     8G  0 part
├─sdb2            8:18   0     4G  0 part
├─sdb3            8:19   0    40G  0 part
└─sdb4            8:20   0   1.8T  0 part
sdc               8:32   0   1.8T  0 disk
sdd               8:48   0   1.8T  0 disk
sde               8:64   0   1.8T  0 disk
sdf               8:80   1   7.5G  0 disk
├─sdf1            8:81   1   918M  0 part
└─sdf2            8:82   1   8.5M  0 part
```
NAME: 块设备名
MAJ:MIN: 主要和次要设备号
RM: 设备是否属于可移动设备，如sdb的RM值为1 ，是可移动设备
SIZE: 设备的容量大小信息
RO: 设备是否为只读
TYPE: 设备是磁盘还是磁盘上的一个分区
MOUNTPOINT: 设备的挂载点

sda 是盘 sda1是区


```shell
lsblk -m # 显示设备所有者相关的信息，包括文件的所属用户、所属组以及文件系统挂载的模式

[root@gogogo ~]# lsblk -m
NAME   SIZE OWNER GROUP MODE
vdb     40G root  disk  brw-rw----
vda     20G root  disk  brw-rw----
└─vda1  20G root  disk  brw-rw----
```

### blkid 
显示关于块的信息

```bash
blkid # 显示所有可用信息 和唯一识别id
blkid /dev/sad1 # 特指
blkid -U # 设备
blkid -g # 清除缓存
```



### lscpu
lscpu 命令可以获得 CPU 的详细信息，如CPU架构（如 i686 或 x86_64）、运算模式（32bit 或 64bit）、核心数、每颗核心的线程数、型号、主频等。

### lsmod
如果你新增加了硬件，却不能被系统自动识别。可能你需要手动地加载对应的内核模块。内核模块安装在 /lib/modules/ 的子目录下，该子目录的名字对应内核的版本。即 /lib/modules/4.10.0-19-generic 目录下包含了 4.10.0-19-generic 内核的驱动程序
```bash
[root@gogogo ~]# lsmod
Module                  Size  Used by
crct10dif_pclmul       16384  1
crc32_pclmul           16384  0
ghash_clmulni_intel    16384  0
aesni_intel           372736  0
cirrus                 16384  0
crypto_simd            16384  1 aesni_intel
cryptd                 24576  2 crypto_simd,ghash_clmulni_intel
glue_helper            16384  1 aesni_intel
drm_kms_helper        176128  3 cirrus
joydev                 24576  0
input_leds             16384  0
drm                   487424  3 drm_kms_helper,cirrus
input_leds             16384  0
drm                   487424  3 drm_kms_helper,cirrus
virtio_balloon         24576  0
```
获取已加载模块的信息，可以使用modinfo命令
```bash
modinfo drm

[root@gogogo ~]# modinfo drm
filename:       /lib/modules/5.3.0-1.el7.elrepo.x86_64/kernel/drivers/gpu/drm/drm.ko
license:        GPL and additional rights
description:    DRM shared core routines
author:         Gareth Hughes, Leif Delgass, José Fonseca, Jon Smirl
license:        GPL and additional rights
description:    DRM bridge infrastructure
author:         Ajay Kumar <ajaykumar.rs@samsung.com>
license:        GPL and additional rights
```
另外，还可以使用 modprobe 命令加载模块，rmmod 命令移除模块。

## 网络
### netstat命令可以帮助查找本机的网络状态
```bash
netstat -pt #可以输出PID以及程序名
```

输出结果
```bash
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address Foreign Address State PID/Program name
tcp 0 0 localhostname:16061 192.168.1.1:50060       ESTABLISHED 22000/java
```

是本机的16061端口在调192.168.1.1:50060上的服务，且本机16061端口上跑的是一个java程序，进程ID是22000

```bash
netstat -ant 
netstat -input
```

### traceroute 路由追踪
traceroute可以追踪网络数据包，并展现路由途径

通过traceroute我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径。当然每次数据包由某一同样的出发点（source）到达某一同样的目的地(destination)走的路径可能会不一样，但基本上来说大部分时候所走的路由是相同的。linux系统中，我们称之为traceroute,在MS Windows中为tracert。 traceroute通过发送小的数据包到目的设备直到其返回，来测量其需要多长时间。一条路径上的每个设备traceroute要测3次。输出结果中包括每次测试的时间(ms)和设备的名称（如有的话）及其IP地址

https://tools.ipip.net/traceroute.php


**使用**
首先你要下载traceroute
yum install traceroute -y

```bash
traceroute [参数] 主机
```


#### 实战
```bash
[root@gogogo ~]# traceroute www.baidu.com
traceroute to www.baidu.com (180.101.49.11), 30 hops max, 60 byte packets
 1  10.0.0.130 (10.0.0.130)  1.151 ms  1.582 ms  2.369 ms
 2  10.255.106.1 (10.255.106.1)  1.056 ms  1.647 ms  2.266 ms
 3  10.128.0.1 (10.128.0.1)  0.441 ms  0.421 ms  0.389 ms
 4  115.231.100.65 (115.231.100.65)  2.023 ms  2.055 ms  2.201 ms
 5  10.100.128.21 (10.100.128.21)  1.844 ms  1.977 ms  2.696 ms
 6  172.31.110.249 (172.31.110.249)  0.632 ms  0.645 ms  0.630 ms
 7  172.31.123.53 (172.31.123.53)  0.536 ms  0.685 ms  0.623 ms
 8  * * *
 9  61.164.31.120 (61.164.31.120)  2.390 ms 220.191.194.116 (220.191.194.116)  3.258 ms 61.164.31.122 (61.164.31.122)  2.264 ms
10  220.191.200.201 (220.191.200.201)  6.861 ms 220.191.200.217 (220.191.200.217)  6.413 ms 220.191.200.201 (220.191.200.201)  5.955 ms
11  202.97.76.10 (202.97.76.10)  13.005 ms 202.97.75.174 (202.97.75.174)  14.885 ms 202.97.22.6 (202.97.22.6)  19.543 ms
12  58.213.95.118 (58.213.95.118)  19.171 ms 58.213.95.2 (58.213.95.2)  14.327 ms 58.213.94.118 (58.213.94.118)  15.157 ms
13  * 58.213.95.126 (58.213.95.126)  20.922 ms 58.213.95.134 (58.213.95.134)  16.233 ms
14  58.213.96.102 (58.213.96.102)  14.907 ms 58.213.96.94 (58.213.96.94)  15.467 ms 58.213.96.90 (58.213.96.90)  15.166 ms
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * * 
```
最左边的是表示每一条的序号,从1开始，每一个序号表示一跳，每一跳表示一个网关 
第二个字段表示主机名和对应ip
后面三个表示发送的三个包所返回的时间
一行*号 有可能是防火墙封掉了ICMP的返回信息，所以我们得不到什么相关的数据包返回数据。

```bash
 # 添加每次发送的数据包个数  
 traceroute -q 4  www.baidu.com

 # 不查主机名，仅显示ip地址
 traceroute -n www.baidu.com

 # 设置跳数数量
 traceroute -m 10 www.baidu.com

 # 发送到直连主机
 traceroute -r www.baidu.com
```

####  工作原理
Traceroute程序的设计是利用ICMP及IP header的TTL（Time To Live）栏位（field）。首先，traceroute送出一个TTL是1的IP datagram（其实，每次送出的为3个40字节的包，包括源地址，目的地址和包发出的时间标签）到目的地，当路径上的第一个路由器（router）收到这个datagram时，它将TTL减1。此时，TTL变为0了，所以该路由器会将此datagram丢掉，并送回一个「ICMP time exceeded」消息（包括发IP包的源地址，IP包的所有内容及路由器的IP地址），traceroute 收到这个消息后，便知道这个路由器存在于这个路径上，接着traceroute 再送出另一个TTL是2 的datagram，发现第2 个路由器...... traceroute 每次将送出的datagram的TTL 加1来发现另一个路由器，这个重复的动作一直持续到某个datagram 抵达目的地。当datagram到达目的地后，该主机并不会送回ICMP time exceeded消息，因为它已是目的地了，那么traceroute如何得知目的地到达了呢？

Traceroute在送出UDP datagrams到目的地时，它所选择送达的port number 是一个一般应用程序都不会用的号码（30000 以上），所以当此UDP datagram 到达目的地后该主机会送回一个「ICMP port unreachable」的消息，而当traceroute 收到这个消息时，便知道目的地已经到达了。所以traceroute 在Server端也是没有所谓的Daemon 程式。

Traceroute提取发 ICMP TTL到期消息设备的IP地址并作域名解析。每次 ，Traceroute都打印出一系列数据,包括所经过的路由设备的域名及 IP地址,三个包每次来回所花时间。

每次写到TTL值访问路由，每到达一个路由，TTL值减少1 到0返回，  重新装载比上次+1的TTL值访问路由，如此循环

<font color='red'>记住他只记录路由, 如果是二层之间的机器访问是不是只有一条？</font>


### mtr 路由追踪
MTR是Linux平台上一款非常好用的网络诊断工具，集成了traceroute、ping、nslookup的功能，用于诊断网络状态非常有用。下面请看简单介绍。
一旦你运行mtr ，它将探测本地系统和你指定的远程主机之间的网络连接。 它首先建立主机之间的每个网络跳跃点（网桥，路由器和网关等）的地址，然后ping每个跳跃点（发送一个序列ICMP ECHO请求）以确定到每台机器的链路的质量。

使用方法
```bash
mtr ip或域名
                                                                   My traceroute  [v0.85]
PG-VPN (0.0.0.0)                                                                                                                   Wed Oct  9 18:22:54 2019
Resolver: Received error response 2. (server failure)er of fields   quit
                                                                                                                   Packets               Pings
 Host                                                                                                            Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. 115.231.97.1                                                                                             0.0%    20    0.8   1.0   0.8   1.3   0.0
 2. 10.100.129.24                                                                                           0.0%    20    0.9   0.9   0.9   1.0   0.0
 3. 10.100.128.65                                                                                            0.0%    20    0.9   1.0   0.9   1.1   0.0
 4. 10.100.129.27                                                                                            0.0%    20    1.3   1.1   1.0   1.3   0.0
 5. 124.14.9.238                                                                                              0.0%    20    4.0   4.1   3.8   4.3   0.0
```


第一列(Host) : IP地址和域名 按n键可以切换IP和域名
第二列(Loss%): 丢包率
第三列(Snt): 设置每秒发送数据包的数量，默认是10 可以通过参数-c来指定
第四列(Last)： 最近一次的Ping值
第五六七列(AVG BEST WRST) : 分别，是Ping的平均 最好 最差值
第八列(StDev): 标准偏差，越大说明相应节点越不稳定。

![2020-01-15-18-12-14](http://img.noback.top/2020-01-15-18-12-14.png)

判别
1. 判断各区域是否存在异常，并根据各区域的情况分别处理。
    区域 A：客户端本地网络，即本地局域网和本地网络提供商网络。针对该区域异常，客户端本地网络相关节点问题，请对本地网络进行排查分析；本地网络提供商网络相关节点问题，请向当地运营商反馈。
    区域 B：运营商骨干网络。针对该区域异常，可根据异常节点 IP 查询归属运营商，然后直接或通过阿里云售后技术支持，向相应运营商反馈问题。
    区域 C：目标服务器本地网络，即目标主机归属网络提供商网络。针对该区域异常，需要向目标主机归属网络提供商反馈问题。
2. 查看节点丢包率，若 Loss% 不为零，则说明这一跳网络可能存在问题。
    导致节点丢包的原因通常有两种：
    人为限制了节点的 ICMP 发送速率，导致丢包。
    节点确实存在异常，导致丢包
   
 命令
```bash
-h  #提供帮助命令
-v  #显示mtr的版本信息
-r  #已报告模式显示
-s  #用来指定ping数据包的大小
-i  #使用这个参数来设置ICMP返回之间的要求默认是1秒
-4  #默认IPv4
-6  # 指定IPv6
-c --report-cycles COUNT # 指定发送多少个数据包
-r --report # 结果显示，并不动态显示
-n --no-dns #不对IP地址做域名解析
-p --split # 将每次追踪的结果分别列出来，而非如 --report 统计整个结果
-a #来设置发送数据包的IP地址 这个对一个主机由多个IP地址是有用的
```

#### 其他用法
```bash
$ mtr -h  #提供帮助命令
$ mtr -v  #显示mtr的版本信息
$ mtr -r  #已报告模式显示
$ mtr -s  #用来指定ping数据包的大小
$ mtr --no-dns  #不对IP地址做域名解析
$ mtr -a  #来设置发送数据包的IP地址 这个对一个主机由多个IP地址是有用的
$ mtr -i  #使用这个参数来设置ICMP返回之间的要求默认是1秒
$ mtr -4  #IPv4
$ mtr -6  #IPv6
```


### 域名查询工具 dig
常用的域名查询工具，可以用来测试域名系统工作是否正常

```bash
dig (选项)(参数)


alpaca@hzj  ~  dig www.baidu.com

; <<>> DiG 9.10.6 <<>> www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24008
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.baidu.com.                 IN      A

;; ANSWER SECTION:
www.baidu.com.          582     IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       227     IN      A       180.101.49.11
www.a.shifen.com.       227     IN      A       180.101.49.12

;; Query time: 8 msec
;; SERVER: 192.168.5.11#53(192.168.5.11)
;; WHEN: Fri Nov 15 15:31:34 CST 2019
;; MSG SIZE  rcvd: 101
```

## 监听

### lsof可以查看系统打开的文件
这里的“文件”包括/proc文件、磁盘文件、网络IO等
```bash
lsof /etc/passwd # 查看哪个进程在占用/etc/passwd
lsof -c mysql  # 列出某个程序打开的文件信息
lsof -u username # 列出某个用户打开的文件
lsof -u ^root # 排除某个用户外的被打开的文件信息
lsof -p 1 # 列出某个进程号打开的文件
lsof -i # 列出所有的网络连接
lsof -i :22 # 列出谁在使用22端口
lsof | grep usbserial #查看usb端口使用情况
```

### ps 查找与进程相关的PID号
```bash
ps a #  显示所有进程以及相关的pid
ps -A # 显示所有进程信息
ps -u root #显示root进程用户信息
ps -ef # 显示所有命令，连带命令行
ps -aux  # 显示所有包含其他使用者的行程
ps c # 列出程序显示每个程序真正的指令名称，而不包含路径，参数或常驻服务的标示
ps -e # 显示每一个程序的环境变量
ps -ef | grep java | grep -v grep # 显示所有a进程,去掉当前的grep进程
kill -9 pid  # 杀死进程
```

## 文件

### mkdir 创建目录
```bash
mkdir (选项) (参数)
-m 建立目标的同时设置属性
-p 多层级建立
-v 显示建立信息 
```

使用
```bash
[root@test-ceph ~]# mkdir -m 777 -pv kk/kk/kkk
mkdir: 已创建目录 "kk"
mkdir: 已创建目录 "kk/kk"
mkdir: 已创建目录 "kk/kk/kkk" 
[root@test-ceph alpaca]# mkdir -pv  ~/alpaca/tt/{,name/}{n,a,m,e}
mkdir: 已创建目录 "/root/alpaca/tt"
mkdir: 已创建目录 "/root/alpaca/tt/n"
mkdir: 已创建目录 "/root/alpaca/tt/a"
mkdir: 已创建目录 "/root/alpaca/tt/m"
mkdir: 已创建目录 "/root/alpaca/tt/e"
mkdir: 已创建目录 "/root/alpaca/tt/name"
mkdir: 已创建目录 "/root/alpaca/tt/name/n"
mkdir: 已创建目录 "/root/alpaca/tt/name/a"
mkdir: 已创建目录 "/root/alpaca/tt/name/m"
mkdir: 已创建目录 "/root/alpaca/tt/name/e"
```

### wget下载文件工具
Linux系统中的wget是一个下载文件的工具，它用在命令行下。对于Linux用户是必不可少的工具，我们经常要下载一些软件或从远程服务器恢复备份到本地服务器。wget支持HTTP，HTTPS和FTP协议，可以使用HTTP代理。所谓的自动下载是指，wget可以在用户退出系统的之后在后台执行。这意味这你可以登录系统，启动一个wget下载任务，然后退出系统，wget将在后台执行直到任务完成，相对于其它大部分浏览器在下载大量数据时需要用户一直的参与，这省去了极大的麻烦。
wget 可以跟踪HTML页面上的链接依次下载来创建远程服务器的本地版本，完全重建原始站点的目录结构。这又常被称作”递归下载”。在递归下载的时候，wget 遵循Robot Exclusion标准(/robots.txt). wget可以在下载的同时，将链接转换成指向本地文件，以方便离线浏览。
wget 非常稳定，它在带宽很窄的情况下和不稳定网络中有很强的适应性.如果是由于网络的原因下载失败，wget会不断的尝试，直到整个文件下载完毕。如果是服务器打断下载过程，它会再次联到服务器上从停止的地方继续下载。这对从那些限定了链接时间的服务器上下载大文件非常有用

```bash
wget (参数) (地址)

-V 显示版本
-h 语法帮助
-b 启动后转入后台执行
-o --output-file=file 把下载记录写到文件中
-a 追加写到文件中
-q 静默输出
-i --input-file=file 下载在file文件中出现过的urls
-F 把输入文件当作html格式来对待
-O 下载并以不同的文件名保存
```

使用
```bash
# 下载一个包
wegt  http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 下载一个文件并重命名，默认以最后一个/后面的内容为文件名
wegt -O test.tar.gz http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz 
# 限速下载
wget --limit-rate=300k http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 断点续传
wget -c http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 后台运行 , 并使用tail观察进度
wget -b http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
tail -f wget-log
# 指定下载文件存放地址
wget --directory-prefix/sources http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 伪装代理名称
wget --user-agent="Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.16 (KHTML, like Gecko) Chrome/10.0.648.204 Safari/534.16" http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 测试下载
wget --spider http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 增加重试下载次数
wget --tries=40 http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 多链接下载
wget -i urllist.txt
# 过滤格式下载
wget --reject=gif http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 把下载信息存入日志文件
wget -o downloads.log http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 限制下载文件大小,超过5m就退出，对单个下载无用，需要递归下载
wget -Q5m -i urllist.txt
# 下载指定格式文件
wget -r -A .pdf http://downloads.sourceforge.net/project/tcl/Tcl/8.6.3/tcl8.6.3-src.tar.gz
# 下载ftp上的内容
wget ftp-url
wget --ftp-user=username --ftp-passwd=passwd xxx.ftp
```

### tar解压缩命令
解压缩命令
参数
```bash
-c #新建打包文件
-t #查看打包文件的内容含有哪些文件名
-x #解压缩或者解打包的功能
-j #通过bzip2的支持进行压缩/解压缩
-z #通过gzip的支持进行压缩/解压缩 
-v #显示压缩和解压缩文件
-f filename # filename为要处理的文件
-C dir # 指定压缩/解压缩的目录dir
```
使用
```bash
压缩：tar -jcv -f filename.tar.bz2 #要被处理的文件或目录名称
查询：tar -jtv -f filename.tar.bz2
解压：tar -jxv -f filename.tar.bz2 #-C 欲解压缩的目录 
```




### find 查找文件
用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。如果使用该命令时，不设置任何参数，则find命令将在当前目录下查找子目录与文件。并且将查找到的子目录和文件全部进行显示

参数
```bash
find [path] [command] [file-path]
-print # 将匹配到的文件名输出到标准输出中
-exec # 找到文件后，对文件列表中的每一个文件都执行一遍exec 后面的shell命令
# 格式如下   find / -name "*.txt" -exec head 10 {} \; 这里{} 一个占位符表示文件列表中的文件， \; 表示shell命令结束
-ok # 与exec 一样，在执行前进行确认
-name filename  #指定名字
-o   # 满足and
-path # 指定路径
! # 取反
-type # 指定参数类型  
#   f 普通文件
#   l 软链接
#   d 目录
#   c 字符设备
#   b 快设备
#   s 套接字
-maxdepth # 定义目录最大深度
-mindepth # 定义目录最小深度 
-size # 指定大小
-owner # 指定拥有者
-depth -d # 使查找在进入子目录前先行查找完本目录
```
使用
```bash
find .  # 列出当前目录以及子目录下所有文件和文件夹
find /home -name "*.txt" #列出/home目录下所有.txt结尾的文件名
find /home -iname "*.txt" #同上，忽略大小写
find /home \(-name "*.txt" -o -name "*.pdf"\) # 满足逻辑关系 and
find /home -name "*.txt" -o -name "*.pdf" # 同上
find /usr/ -path "*local" # 查找文件或路径
find . -regex ".*\(\.txt\|\.pdf\)$" # 正则匹配
find /home ! -name "*.txt" # ! 否定
find . -type 参数类型
find . -maxdepth 3 -type f # 定义目录最大深度 3
find . -mindepth 2 -type f # 定义目录最小深度 2
find . -type f -atine -7 # 最近7天内
find . -type f -atime 7 # 7天前当天被访问过的文件
find . -type f -atime +7 # 7天前被访问过的文件
find /var/log -size +1G # 查找大于1G的文件
find /data -owner user # 找到user用户的文件
find ./ -iname '_macosx' -depth -exec rm -rf {} \; #  删除自动生成的文件 {} 表示匹配到的文件，\;表示shell命令结束
```
https://wangchujiang.com/linux-command/c/find.html


###  cp 复制文件 
```bash
cp (选项) (参数) 

-r 可以复制目录
-i 覆盖文件之前询问
-f 强行复制文件或目录,无论是否存在
-u 使用这项参数后只会在源文件的更改时间较目标文件更新时或是名称相互对应的目标文件并不存在时，才复制文件；
-v 看详细信息
```

### ldd 查看动态库依赖关系
首先ldd并不是一个二进制的可执行程序，他是依附在Linux中的一个shell脚本
用于打印程序或者库文件所依赖的共享库列表
比如
https://www.cnblogs.com/zhangjxblog/p/7776556.html
https://blog.csdn.net/qq_26819733/article/details/50610129
```bash
[root@gogogo ~]# ldd /bin/ls
        linux-vdso.so.1 =>  (0x00007ffe7cbcc000)
        libselinux.so.1 => /lib64/libselinux.so.1 (0x00007fedc8e77000)
        libcap.so.2 => /lib64/libcap.so.2 (0x00007fedc8c72000)
        libacl.so.1 => /lib64/libacl.so.1 (0x00007fedc8a69000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fedc869b000)
        libpcre.so.1 => /lib64/libpcre.so.1 (0x00007fedc8439000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007fedc8235000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fedc909e000)
        libattr.so.1 => /lib64/libattr.so.1 (0x00007fedc8030000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fedc7e14000)
```

### alias 查看快捷输入
alias 或者 ~/.bash_profile


### ls 查看当前目录文件
```bash
# 语法
ls [option] [name]
```
-a 显示所有文件或目录
-l 除文件名称外，亦将文件型态、权限、拥有者、文件大小等资讯详细列出
-r 倒叙显示
-t 时间顺序排列
-R 递归显示
-h 以可读大小显示
-i 额外显示inode信息

```bash
# 常用
ls -al
ls -altr #倒叙时间
ls -lh # 列出详细信息并以可读大小显示文件大小
```

### du查看目录或文件所占用磁盘大小
```bash
# 语法
du [option] [name] 
```
-h 以人类可读的方式显示
-a：显示目录占用的磁盘空间大小，还要显示其下目录和文件占用磁盘空间的大小
-s：显示目录占用的磁盘空间大小，不要显示其下子目录和文件占用的磁盘空间大小
-c：显示几个目录或文件占用的磁盘空间大小，还要统计它们的总和
--apparent-size：显示目录或文件自身的大小
-l ：统计硬链接占用磁盘空间的大小
-L：统计符号链接所指向的文件占用的磁盘空间大小　

```bash
# 常用
du -sh 查看当前目录总共占的容量。而不单独列出各子项占用的容量 
du -lh --max-depth=1 : 查看当前目录下一级子文件和子目录占用的磁盘容量。

du -sh * | sort -n 统计当前文件夹(目录)大小，并按文件大小排序
du -sk filename 查看指定文件大小 
```

### 文件传输  scp命令
```bash
scp -h # 查看帮助
scp -v # 显示进度
scp -P # 选择端口
scp -r # 传输目录
```

1. 从本地将文件传输到服务器
scp【本地文件的路径】【服务器用户名】@【服务器地址】：【服务器上存放文件的路径】
scp /Users/mac_pc/Desktop/test.png root@192.168.1.1:/root

2. 从本地将文件夹传输到服务器
scp -r【本地文件的路径】【服务器用户名】@【服务器地址】：【服务器上存放文件的路径】
sup -r /Users/mac_pc/Desktop/test root@192.168.1.1:/root

3. 将服务器上的文件传输到本地
scp 【服务器用户名】@【服务器地址】：【服务器上存放文件的路径】【本地文件的路径】
scp root@192.168.1.1:/data/wwwroot/default/111.png /Users/mac_pc/Desktop

4. 将服务器上的文件夹传输到本地
scp -r 【服务器用户名】@【服务器地址】：【服务器上存放文件的路径】【本地文件的路径】
sup -r root@192.168.1.1:/data/wwwroot/default/test /Users/mac_pc/Desktop







## 命令

### xargs命令
xargs 将标准输入数据转换成命令行参数
<font color='red'>xargs默认命令是echo，空格是默认的定界符</font>
如果下面的一个命令一定要使用到参数，但是又不接受管道，我们可以使用xargs
比如我们要打包当前目录下面的文件
```bash
find . -type f  | xargs tar -cvzf output.tar.gz
```

参数
```bash
-n 输出时候指定一定数量的xargs输出
-d 自定义一个定界符
-I 使用占位符 xargs可以将输入的参数通过-i {} 指定占位符，来格式化输出内容
-r 当xargs的输入为空的时候则停止xargs，不用再去执行后面的命令了，-r是xargs的默认选项。
-e 指定标签flag 即当xargs分隔到指定的flag的时候，就会停止分隔 
  注意这里的`flag可以是一个字符串或者是由空格分隔的多个字符串，当xargs分析到这个flag时，就会停止工作`

-p：当每次执行一个argument的时候询问一次用户。
-t：表示先打印命令，然后再执行
--max-procs ： 指定进程数
```
使用
```bash
cat test.txt

a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z 
# 多行输入单行输出
cat test.txt | xargs
a b c d e f g h i j k l m n o p q r s t u v w x y z
# -n选项多行输出
cat test.txt | xargs -n3
a b c
d e f
g h i
j k l
m n o
p q r
s t u
v w x
y z
# -d选项可以定义一个定界符
echo "nameXnameXnameXname" | xargs -dX
name name name name

# 指定结束符号，先打印命令行再输出
echo a bc, d e rf f | xargs -ebc, -t
echo a
a


[root@node test]# ls | xargs -I '{}' echo {}.md
ddq.md
ee.md
ii.md
qqw.md
rr.md
tt.md
uu.md
xx.md
yy.md
```
在xargs的使用上，你可以将它默认成先把所有的内容都整理到一行上面来,然后根据参数的不同来整理内容