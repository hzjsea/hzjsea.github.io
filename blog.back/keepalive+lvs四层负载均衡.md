---
title: keepalive+lvs 四层负载均衡
categories:
  - linux
tags:
  - keepalive
  - lvs
  - linux
abbrlink: a33a1a7d
date: 2020-01-05 13:06:09
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [keepalive+lvs](#keepalivelvs)
  - [关键词](#关键词)
  - [仅使用lvs](#仅使用lvs)
  - [仅使用keepalive](#仅使用keepalive)
  - [keepalive+lvs 主从方式高可用](#keepalivelvs-主从方式高可用)
    - [主从模式](#主从模式)
    - [双主方式](#双主方式)
    - [sorry_server 和local RS](#sorry_server-和local-rs)
    - [脑裂解决方法](#脑裂解决方法)
    - [自定义健康状态检测](#自定义健康状态检测)
      - [keepalived自动配置](#keepalived自动配置)
  - [报错解决方案](#报错解决方案)
  - [概念篇](#概念篇)
    - [配置文件说明](#配置文件说明)
    - [ipvsadm 概念篇](#ipvsadm-概念篇)
    - [其他概念](#其他概念)
  - [网址](#网址)

<!-- /code_chunk_output -->
<!--more-->
# keepalive+lvs

## 关键词
LB (Load Balancer 负载均衡)
HA (High Available 高可用)
Failover (失败切换)
Cluster (集群)
LVS (Linux Virtual Server Linux 虚拟服务器)
DS (Director Server)，指的是前端负载均衡器节点
RS (Real Server)，后端真实的工作服务器
VIP (Virtual IP)，虚拟的 IP 地址，向外部直接面向用户请求，作为用户请求的目标的 IP 地址
DIP (Director IP)，主要用于和内部主机通讯的 IP 地址
RIP (Real Server IP)，后端服务器的 IP 地址
CIP (Client IP)，访问客户端的 IP 地址


<!-- more -->
## 仅使用lvs

单节点lvs 、 vip  两台RS服务器


```bash
vip: 10.0.5.97/20
lvs-director: 10.0.5.92

RS1: 10.0.5.90
RS2: 10.0.5.91
```

在lvs上进行操作
```bash
# 安装lvs
yum install ipvsadm -y 
# 安装net-tools
yum install net-tools


# 在网卡上绑定vip
ifconfig eno1:0 10.0.5.97 broadcast 10.0.5.97 netmask 255.255.240.0 up
# 添加路由
route add -host 10.0.5.97 dev eno1
# 启用系统的包转发功能
echo "1" > /proc/sys/net/ipv4/ip_forward
# 清空系统之前所有的ipvsadm规则
ipvsadm --clear
ipvsadm -C
# 增加虚拟ip (-s rr 表述轮询)
ipvsadm -A -t 10.0.5.97:8888 -s rr
# ipvsadm -D -t 10.0.5.97:8888 # 删除规则

# 添加服务对象 -g表示工作模式为直接路由 端口必须一致
ipvsadm -a -t 10.0.5.97:8888 -r 10.0.5.91:8888 -g
ipvsadm -a -t 10.0.5.97:8888 -r 10.0.5.92:8888 -g
```

在RS机上进行操作
```bash
# //RS1
# 绑定VIP 
ifconfig eno1:0 10.0.5.91 broadcast 10.0.5.91 netmask 255.255.240.0 up
# 添加路由
route add -host 10.0.5.91 dev eno1:0

#//RS2 同
```

lvs搭建成功后查看搭建状态
```bash
[root@lvs-web1 ~]# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.5.97:8888 rr
  -> 10.0.5.90:8888               Route   1      0          0
  -> 10.0.5.91:8888               Route   1      0          0 
```

当你访问VIP地址的时候，VIP会指向其所服务的两个真实IP地址 在轮询模式下，一定时间后指向另一个ip



<font color='red'>单节点lvs可能会出现错误，需要使用Keepalive+lvs</font>



## 仅使用keepalive
不使用RS服务器，仅使用两台DS服务器
测试vip在两台DS之间保持活性的过程
目的----> 保证vip的活性

```bash
vip 10.0.5.96

DS1: 10.0.5.93
DS2: 10.0.5.90  
```
在DS-Master机器上操作
```bash
# 安装keepalived
yum install keepalived
# 修改配置文件
! Configuration File for keepalived


global_defs {  //全局配置
   notification_email {     //通知邮件， 当keepalived中master和backup身份改变的是否会发送邮件  
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL_master   // 集群名字
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {   //vrrp实例 +  名字(同集群名字需要相同)
    state MASTER   // 当前机器身份 只能有一个master
    interface enp10s0  //当前网络接口
    virtual_router_id 51 // 虚拟路由序号 主从需要相同
    priority 100  // 优先级别 主> 从
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {  //虚拟ip
        10.0.5.96/20
    }
}

```
在DS-Backup机器上的操作
```bash
# 安装keepalived
yum install keepalived
# 修改配置文件
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state BACKUP   // DS-backup机器
    interface eno1 
    virtual_router_id 51
    priority 10  //优先级别
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.5.96/20
    }
} 
```

主从机器启动keepalived后

![2020-01-05-18-04-33](http://img.noback.top/2020-01-05-18-04-33.png)
![2020-01-05-18-04-56](http://img.noback.top/2020-01-05-18-04-56.png)

可以发现现在的vip是在DS-master上的，
为了验证master主机出现错误的时候，vip会漂移到DS-backup的现象，我们可以在master关闭keepalived
![2020-01-05-18-05-44](http://img.noback.top/2020-01-05-18-05-44.png)
![2020-01-05-18-06-06](http://img.noback.top/2020-01-05-18-06-06.png)
以上，当master出现错误的时候，backup机器上出现vip，从而保证vip的活性，以此来保证lvs的稳定性

<font color='blue'>另外，当master机器再次重启的时候，vip又会飘回到master身上</font>

## keepalive+lvs 主从方式高可用

### 主从模式

需要准备四台服务器，均开启httpd服务
![2020-01-05-13-15-33](http://img.noback.top/2020-01-05-13-15-33.png)
![2020-01-05-13-15-53](http://img.noback.top/2020-01-05-13-15-53.png)
![2020-01-05-13-16-08](http://img.noback.top/2020-01-05-13-16-08.png)
![2020-01-05-13-16-21](http://img.noback.top/2020-01-05-13-16-21.png)

配置
```bash
vip 10.0.5.96
DS1: 10.0.5.90 lvs-master
DS2: 10.0.5.93 lvs-slave


RS1: 10.0.5.91 lvs-web1
RS2: 10.0.5.92 lvs-web2 
# 测试
ping lvs-master
```

首先保证 90 93两台机器的keepalive搭建成功，配置参考上面的keepalived


Keepalived主备配置文件区别：

　　　　01. router_id 信息不一致

　　　　02. state 状态描述信息不一致

　　　　03. priority 主备竞选优先级数值不一致



master机器上配置lvs
```bash
vi /etc/keepalived/keepalived.conf

virtual_server 10.0.5.96 8888 {  //vip
    delay_loop 3
    lb_algo rr     // 负载均衡模式 rr--轮询
    lb_kind DR     // 负载均衡连接形式  直连路由
    !persistence_timeout 50 
    protocol TCP 

    real_server 10.0.5.91 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888  //连接端口
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 10.0.5.92 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```
backup机器上配置lvs
```bash
vi /etc/keepalived/keepalived.conf

virtual_server 10.0.5.96 8888 {
    delay_loop 3
    lb_algo rr
    lb_kind DR
    !persistence_timeout 50
    protocol TCP

    real_server 10.0.5.91 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 10.0.5.92 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

RS机器上配置 
<font color='red'>只要是作为lvs服务的对象，这些步骤都要做，包括后面如果我们把ds-master机器在作为keepalived调度机的同时，也提供web服务的时候，就需要做这些步骤</font>

```bash
# 绑定VIP
ifconfig lo:0 10.0.5.96 netmask 255.255.240.0 broadcast 10.0.5.96 up 
# 添加路由
route add -host 10.0.5.96 dev lo:0
# 抑制arp
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce  
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 1 >/proc/sys/net/ipv4/conf/lo/arp_ignore  

## 永久抑制arp
vi /etc/sysctl.conf
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
```


同时启动keepalived，查看lvs连接状态

```bash
systemctl start keepalived
ipvsadm -ln
[root@localhost ~]# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.5.96:8888 rr
  -> 10.0.5.91:8888               Route   1      0          0
  -> 10.0.5.92:8888               Route   1      0          0
  
while 1; do curl http://10.0.5.96:8888/| grep '< h2 >.*  <\/ h2 \>' ;sleep 1; done 
```

可以看到
![2020-01-06-14-48-22](http://img.noback.top/2020-01-06-14-48-22.png)

### 双主方式

互为主从：主从都在工作；其中一个宕机了，VIP漂移到另一个上，提供服务

配置
```bash
vip1: 10.0.5.96
vip2: 10.0.5.97

RS1: 10.0.5.91
RS2: 10.0.5.92

DS1: 10.0.5.90
DS2: 10.0.5.93
```

DS1 配置:

```bash
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL_master
}

vrrp_instance VI_1 {
    state MASTER
    interface enp10s0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.5.96/20
    }
}

virtual_server 10.0.5.96 8888 {
    delay_loop 3
    lb_algo rr
    lb_kind DR
    protocol TCP

    real_server 10.0.5.91 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 10.0.5.92 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
vrrp_instance VI_2 {
    state BACKUP
    interface enp10s0
    virtual_router_id 52
    priority 11
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.5.97/20
    }
}


virtual_server 10.0.5.97 8888 {
    delay_loop 3
    lb_algo rr
    lb_kind DR
    protocol TCP

    real_server 10.0.5.91 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 10.0.5.92 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```


DS2配置

```bash
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP
    interface eno1
    virtual_router_id 51
    priority 10
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.5.96/20
    }
}

virtual_server 10.0.5.96 8888 {
    delay_loop 3
    lb_algo rr
    lb_kind DR
    protocol TCP

    real_server 10.0.5.91 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 10.0.5.92 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface eno1
    virtual_router_id 52
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.5.97/20
    }
}


virtual_server 10.0.5.97 8888 {
    delay_loop 3
    lb_algo rr
    lb_kind DR
    protocol TCP

    real_server 10.0.5.91 8888 {
        weight 1
        TCP_CHECK {
            connect_port 8888
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 10.0.5.92 8888 {
        weight 1
        TCP_CHECK {
        connect_port 8888
        connect_timeout 3
        nb_get_retry 3
        delay_before_retry 3
        }
    }
}
``` 
两个配置都差不多，无非就是两台机器都作为对方机器的Master和Backup机器，同时拥有两个vip，注意权重的大小

RS上的配置
```bash
#配置VIP到本地回环网卡lo上，并只广播自己
ifconfig lo:0 172.17.100.100 broadcast 172.17.100.100 netmask 255.255.255.255 up
ifconfig lo:1 172.17.100.101 broadcast 172.17.100.101 netmask 255.255.255.255 up 
#配置本地回环网卡路由
route add -host 172.17.100.100 lo:0
route add -host 172.17.100.101 lo:1
# 关闭arp
vim /etc/sysctl.conf
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
```

查看结果
![2020-01-07-17-41-28](http://img.noback.top/2020-01-07-17-41-28.png)

关闭其中一个DS机器时，另外一台DS机器上出现三个IP ，自身IP+2个VIP
![2020-01-07-17-42-27](http://img.noback.top/2020-01-07-17-42-27.png)

https://www.cnblogs.com/f-ck-need-u/p/8492298.html#2-keepalived-lvs-


### sorry_server 和local RS
如果RS都在同一刻down掉了的话，外界就无法访问网站了。因为vip已经没有了RS的去处，这里提供了两种解决方法
1. 使用keepalived来配置一个服务页面。例如告诉外界客户端网站正在维护状态.
使用sorry_server 来指向 某一个ip+端口, 因为是在所有RS都宕机的情况下sorry server提供的临时服务才生效，因此通常将sorry server配置在virtual_server中而非real_server中。

需要在master和backup节点上都配置sorry_server
```bash
virtual_server 192.168.100.10 8888 {
        delay_loop 6
        lb_algo wrr
        lb_kind DR
        protocol TCP

        sorry_server 127.0.0.1 8888

# 也可以指向本地，需要开启httpd服务
```
开启keepalived 然后将两台RS的机器上的httpd服务全部停掉，会发现vip会漂移到两台DS中的其中一台上

2. 使用local RS
对于集群系统不大的情况下，DS 一般会比较空闲，这样就比较浪费资源。这时通常会将LVS Director自身也作为一个RS，一边提供web服务，一边提供调度功能，不过应该将它的调度权重设置低一点，以免影响负载均衡的性能。这称为local RS，local RS的RIP可以写Director上的任意地址(127.0.0.1都可以)。例如：
```bash
virtual_server 192.168.100.10 8888 {
    ....

    real_server 127.0.0.1 8888 {
        weight 1
        TCP_CHECK {
                connect_port 8888
                connect_timeout 1
                nb_get_retry 2
                delay_before_retry 1
        }
}
} 
```

<font color='blue'>两个不应该同时配置，因为如果local RS 坏了，sorry server肯定无法调度</font>


### 脑裂解决方法
在高可用（HA）系统中，当联系2个节点的“心跳线”断开时，本来为一整体、动作协调的HA系统，就分裂成为2个独立的个体。由于相互失去了联系，都以为是对方出了故障。两个节点上的HA软件像“裂脑人”一样，争抢“共享资源”、争起“应用服务”，就会发生严重后果——或者共享资源被瓜分、2边“服务”都起不来了；或者2边“服务”都起来了，但同时读写“共享存储”，导致数据损坏（常见如数据库轮询着的联机日志出错）。

1. 添加冗余的心跳线，例如：双线条线（心跳线也HA），尽量减少“裂脑”发生几率；
2. 设置仲裁机制。例如设置参考IP（如网关IP），当心跳线完全断开时，2个节点都各自ping一下参考IP，不通则表明断点就出在本端。不仅“心跳”、还兼对外“服务”的本端网络链路断了，即使启动（或继续）应用服务也没有用了，那就主动放弃竞争，让能够ping通参考IP的一端去起服务。更保险一些，ping不通参考IP的一方干脆就自我重启，以彻底释放有可能还占用着的那些共享资源。

脑裂原因：
1. Keepalived配置里同一 VRRP实例如果 virtual_router_id两端参数配置不一致也会导致裂脑问题发生。
2. RS上开启了 iptables防火墙阻挡了心跳消息传输。
3. 高可用服务器上心跳网卡地址等信息配置不正确，导致发送心跳失败。
4. 其他服务配置不当等原因，如心跳方式不同，心跳广插冲突、软件Bug等

解决:
1. 做好对裂脑的监控报警（如邮件及手机短信等或值班）.在问题发生时人为第一时间介入仲裁，降低损失。例如，百度的监控报警短倍就有上行和下行的区别。报警消息发送到管理员手机上，管理员可以通过手机回复对应数字或简单的字符串操作返回给服务器.让服务器根据指令自动处理相应故障，这样解决故障的时间更短.
2. 强行关闭一个节点保证，避免争夺


<font color='red'>检测当前服务器上是否存在vip</font>

要赋予执行权限
```bash
while true
do
if [ `ip a show eth0 |grep 10.0.0.3|wc -l` -ne 0 ]
then
    echo "keepalived is error!"
else
    echo "keepalived is OK !"
fi
done
```


### 自定义健康状态检测
keepalived可以通过设置vrrp_script自定义
```bash
# 自定义VRRP实例健康检查脚本 keepalived只能做到对自身问题和网络故障的监控，Script可以增加其他的监控来判定是否需要切换主备
vrrp_script chk_sshd {
 
    script "killall -0 sshd"    # 示例为检查sshd服务是否运行中
 
    interval 2         # 检查间隔时间
    weight -4          # 检查失败降低的权重
}
```

#### keepalived自动配置
```bash

# /keepalived.conf
! Configuration File for keepalived

global_defs {
router_id SLB-SAD
}

vrrp_script chk_upyun {
  ¦ ¦ ¦ script "/etc/keepalived/bin/check_vip.sh"
  ¦ ¦ ¦ interval 3
}

vrrp_instance upyun_lb {
  ¦ state MASTER
  ¦ interface eth3
  ¦ virtual_router_id 20
  ¦ priority 100
  ¦ advert_int 1
  ¦ notify_master /etc/keepalived/bin/change_master.sh
  ¦ notify_backup /etc/keepalived/bin/change_backup.sh
  ¦ authentication {
  ¦ ¦ ¦ auth_type PASS
  ¦ ¦ ¦ auth_pass upyun.com
  ¦ }
  ¦ track_script {
  ¦ ¦ ¦ chk_upyun
  ¦ }
  ¦ virtual_ipaddress {
  ¦ ¦ ¦ 192.168.147.20 label eth3:9
  ¦ }
} 

include /etc/keepalived/virserver.conf


# virserver.conf
virtual_server 192.168.147.20 8600 {
  ¦ delay_loop 2
  ¦ lb_algo wrr
  ¦ lb_kind DR
  ¦ protocol UDP
  ¦ #persistence_timeout 5
  ¦ real_server 192.168.13.250 8600 {
  ¦ ¦ ¦ weight 1
  ¦ ¦ ¦ connect_port 8600
  ¦ ¦ ¦ connect_timeout 5
  ¦ ¦ ¦ nb_get_retry 1
  ¦ ¦ ¦ delay_before_retry 1
  ¦ }
}
```

状态检测脚本
```bash
# check_backup 
# 修改状态从master-> backup  权重 100 -> 80  注销 notify_backup 注销include 
echo "--- I am BACKUP #`date`" >> /tmp/keepalived.log
sed -r -i '/state/s^MASTER^BACKUP^g;/priority/s^100^80^g;/notify_backup/s@^@#@g;/include/s@^@#@g;' /etc/keepalived/keepalived.conf
systemctl restart keepalived

# check_master
echo "+++ I am MASTER #`date`" >> /tmp/keepalived.log
sed -r -i '/state/s^BACKUP^MASTER^g;/priority/s^80^100^g;/notify_backup/s@#@@g;' /etc/keepalived/keepalived.conf
ipvsadm --set 100 15 15
systemctl restart keepalived
```







## 报错解决方案
keepalived报错信息在/usr/log/messages下面
1. 主服务器停止后，备用服务没有启用
监控主服务器上的日志
Jun 28 09:18:32 rust Keepalived_vrrp: receive an invalid ip number count
associated with VRID!
Jun 28 09:18:32 rust Keepalived_vrrp: bogus VRRP packet received on eth0 !!!
Jun 28 09:18:32 rust Keepalived_vrrp: VRRP_Instance(VI_1) Dropping received
VRRP packet.HA

解决方案：
改变配置文件/etc/keepalived/keepalived.conf 中virtual_route_id的值
virtual_router_id 60 主从方都要改，默认为51

2. lvs默认超时时间过程，导致框架已经搭建成功，但是效果看不出来
900 120 300这三个数值分别是TCP TCPFINUDP的时间.也就是说一条tcp的连接经过lvs后,lvs会把这台记录保存15分钟，就是因为这个时间过长，所以大部分人都会发现做好LVS DR之后轮询现象并没有发生，而且我也看到大部分的教程是没有说明这一点的，巨坑！！！！！！因为是实验性质，所以将此数值调整为非常小，使用以下命令调整
在两台DS服务器上修改
```bash
[root@DR1 keepalived]# ipvsadm -L --timeout
Timeout (tcp tcpfin udp): 900 120 300 
[root@DR1 ~]# ipvsadm --set 1 2 1
```




## 概念篇
Keepalived软件起初是专为LVS负载均衡软件设计的，用来管理并监控LVS集群系统中各个服务节点的状态，后来又加入了可以实现高可用（HA）的VRRP功能。
于是keepalived除了能够管理LVS软件外，还可以作为其他服务（例如：Nginx、Haproxy、MySQL等）的高可用解决方案软件

Keepalived软件主要是通过VRRP协议实现高可用功能的。 官网https://www.keepalived.org/

主要功能
--> 管理LVS负载均衡
--> 实现LVS集群节点的健康检查
--> 作为系统网络服务的高可用性（failover）

运行流程
在 Keepalived服务正常工作时，主 Master节点会不断地向备节点发送（多播的方式）心跳消息，用以告诉备Backup节点自己还活看，当主 Master节点发生故障时，就无法发送心跳消息，备节点也就因此无法继续检测到来自主 Master节点的心跳了，于是调用自身的接管程序，接管主Master节点的 IP资源及服务。而当主 Master节点恢复时，备Backup节点又会释放主节点故障时自身接管的IP资源及服务，恢复到原来的备用角色。


### 配置文件说明
全局配置，一般保留路由标识信息就可以了
```bash
global_defs {    #全局配置

    notification_email {   #定义报警邮件地址
      acassen@firewall.loc
      failover@firewall.loc  # 收件人
      sysadmin@firewall.loc
    } 
    notification_email_from Alexandre.Cassen@firewall.loc  #定义发送邮件的地址
    smtp_server 192.168.200.1   #邮箱服务器 
    smtp_connect_timeout 30      #定义超时时间
    router_id LVS_DEVEL        #定义路由标识信息，相同局域网唯一
 } 
```
虚拟ip配置 brrp
```bash
vrrp_instance VI_1 {   #定义实例
    state MASTER         #状态参数 master/backup 只是说明
    interface eth0       #虚IP地址放置的网卡位置
    virtual_router_id 51 #vrrp_instance的唯一标识
    priority 100         # keepalived权重,数值越大权重越大,MASTER应大于BACKUP
    advert_int 1        #发送心跳间隔,如果backup1秒收不到心跳就接管,单位是秒
    authentication {     # ↓
        auth_type PASS    #↓
        auth_pass 1111    #认证
    }                        #↑ 
    virtual_ipaddress {  #↓
        192.168.200.16    #vip
        192.168.200.17
        192.168.200.18
    }
    nopreempt # 设置不抢占功能 
              #nopreempt 表示主节点故障恢复后不再切回到主节点，让服务一直在备用节点下工作，直到备用节点出现故障才会进行切换。
    preemtp_delay 300  # 设置抢占延时时间，单位是秒。
}  
```
lvs配置
```bash
#相当于 ipvsadm -A -t 192.168.0.89:80 -s wrr 
virtual_server 192.168.0.89 80 {
    delay_loop 6             #服务健康检查周期,单位是秒
    lb_algo wrr                 #调度算法
    lb_kind DR                 #模式
    nat_mask 255.255.255.0   
    persistence_timeout 50   #回话保持时间,单位是秒
    protocol TCP             #TCP协议转发

    sorry_server <IPADDR> <PORT> # 当所有 real server 失效后，指定的 Web 服务器的虚拟主机地址。
#添加后端realserver
#相当于 ipvsadm -a -t 192.168.0.89:80 -r 192.168.0.93:80 -w 1
    real_server 192.168.0.93 80  {    #realserver的真实IP
        weight 1                      #权重
        #健康检查
        TCP_CHECK {
            connect_timeout 8         #超时时间
            nb_get_retry 3            #重试次数
            delay_before_retry 3      #重试间隔
            connect_port 80           #检查realserver的80端口,如果80端口没监听,就会从集群中剔除
        }
    }
    real_server 192.168.0.94 80  {
        weight 1
        TCP_CHECK {
           connect_timeout 8
           nb_get_retry 3
           delay_before_retry 3
           connect_port 80
        }
    }

} 
```



### ipvsadm 概念篇

```bash
添加虚拟服务器
    语法:ipvsadm -A [-t|u|f]  [vip_addr:port]  [-s:指定算法]
    -A:添加
    -t:TCP协议
    -u:UDP协议
    -f:防火墙标记
    -D:删除虚拟服务器记录
    -E:修改虚拟服务器记录
    -C:清空所有记录
    -L:查看
添加后端RealServer
    语法:ipvsadm -a [-t|u|f] [vip_addr:port] [-r ip_addr] [-g|i|m] [-w 指定权重]
    -a:添加
    -t:TCP协议
    -u:UDP协议
    -f:防火墙标记
    -r:指定后端realserver的IP
    -g:DR模式
    -i:TUN模式
    -m:NAT模式
    -w:指定权重
    -d:删除realserver记录
    -e:修改realserver记录
    -l:查看
通用:
    ipvsadm -ln:查看规则
    service ipvsadm save:保存规则 
```



<font color='red'>lvs的几种调度算法</font>

- RR：roundrobin
轮询,后端RS均摊所有的请求
- WRR weighted RR
加权轮询，根据权值来分配请求的数量
- SH：Source Hashin[root@lvs-web1 ~]# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.5.97:8888 rr
  -> 10.0.5.90:8888               Route   1      0          0
  -> 10.0.5.91:8888               Route   1      0          0g
- DH：Destination Hashing
- LC：least connections
- WLC：Weighted LC

[具体的调度算法](https://blog.51cto.com/191226139/2090128)


### 其他概念
<font color='red'>什么是VRRP</font>
VRRP是Virtual Router RedundancyProtocol(虚拟路由器冗余协议）的缩写，VRRP出现的目的就是为了解决静态路由单点故障问题的，它能够保证当个别节点宕机时，整个网络可以不间断地运行。VRRP通过一种竞选机制来将路由的任务交给某台VRRP路由器的。
https://www.cnblogs.com/f-ck-need-u/p/8483807.html

<font color='red'>keepalived官网</font>
http://www.keepalived.org

<font color='red'>keepalived高可用架构图</font>
![2020-01-07-15-29-24](http://img.noback.top/2020-01-07-15-29-24.png)

## 网址

高并发场景 LVS 安装及高可用实现
https://www.cnblogs.com/clsn/p/7920637.html#auto_id_30
VRRP原理和分析
https://www.jianshu.com/p/4b46586e79aa
Haproxy+Keepalived高可用环境部署梳理（主主和主从模式）
https://blog.51cto.com/dengaosky/2129856

https://www.cnblogs.com/f-ck-need-u/p/8492298.html#3-keepalived-lvs-
Keepalived+LVS的高可用集群网站架构
https://www.cnblogs.com/along21/p/7841132.html#auto_id_6