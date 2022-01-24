---
title: linux命令iptables
date: 2020-11-26 09:53:25   
categories: [linux]  
tags: [linux,linuxcc]
---


<!-- more -->


# linux命令iptables
netfilter/iptables（简称为iptables）组成Linux平台下的包过滤防火墙，与大多数的Linux软件一样，这个包过滤防火墙是免费的，它可以代替昂贵的商业防火墙解决方案，完成封包过滤、封包重定向和网络地址转换（NAT）等功能。


## iptables原理
iptables搭建在linux服务器上,主要工作在网络层,处理网络层的数据包,上下连通传输层和数据链路层. iptables是一种"用户态"的防火墙,管理员定义一系列规则来过滤数据包中的内容.


## iptables
防火墙工具,iptables的由三部分组成,分别是`表`,`链`以及`规则`
```bash
# 层级区分为
Tables -> Chains -> Rules
```
其中表分为4种类型的表,分别为`fileter`,`nat`,`mangle`,`raw`,四种表没有执行前后的区分不同的表也具有对应的操作链,大致结构图如下
![2020-11-26-10-36-18](http://noback.upyun.com/2020-11-26-10-36-18.png)

> Filter表

Filter表示iptables的默认表,他具有以下三种内建链
- INPUT链 – 处理来自外部的数据。
- OUTPUT链 – 处理向外发送的数据。
- FORWARD链 – 将数据转发到本机的其他网卡设备上。

FORWARD链需要开启权限,默认情况下，Linux的三层包转发功能是关闭的，如果要让我们的linux实现转发，则需要打开这个转发功能，可以 改变它的一个系统参数，使用`sysctl net.ipv4.ip_forward=1`或者`echo "1" > /proc/sys/net/ipv4/ip_forward`命令打开转发功能

> NAT表

- PREROUTING链 – 处理刚到达本机并在路由转发前的数据包。它会转换数据包中的目标IP地址（destination ip address），通常用于DNAT(destination NAT)。
- POSTROUTING链 – 处理即将离开本机的数据包。它会转换数据包中的源IP地址（source ip address），通常用于SNAT（source NAT）。
- OUTPUT链 – 处理本机产生的数据包。

> Mangle表 
> mangle表的主要功能是根据规则修改数据包的一些标志位，以便其他规则或程序可以利用这种标志对数据包进行过滤或策略路由.
> 策略路由: 因为mangel是数据包进入的时候,经历过的第一张表, 第一个链是mangle表中的PREROUTING,因此可以在数据包进入的时候就对这些数据包打上标签,后面使用iprouter对不同的包做不同的路由选择


Mangle表用于指定如何处理数据包。它能改变TCP头中的QoS位。Mangle表具有5个内建链
- PREROUTING  - 数据包经过的第一个链,接收原有数据包内容,对其做本机器操作初始化,例如打标签
- OUTPUT
- FORWARD
- INPUT
- POSTROUTING

> Raw表用于处理异常

- PREROUTING chain
- OUTPUT chain


> iptables整个架构是由四表5链组成的,常用的有Filter,nat 以及mangle表
> 规则表之间的优先顺序 Raw——mangle——nat——filter



## 数据包在iptables中的走向
iptables中有多种类型的表,每种类型的表又具有不同类型的链,因此需要先了解数据包在iptables的走向,这样才能在指定的位置对数据包做拦截或者过滤
![2020-11-26-10-50-48](http://noback.upyun.com/2020-11-26-10-50-48.png)

- 数据包通过网口到达服务器,进入nat表的`PREROUTING`
- 内核判断,如果不是本机则直接经过Filter的`FORWARD`出去
- 如果是本机的数据包,则进入Filter的`INPUT`做处理
- 最后交给Nat的`OUTPUT`做处理
- 处理完毕后做发送操作,交给Filter的`OUTPUT`链做处理
- 最后交给Nat的`POSTROUTING`做离开处理

## iptables中的链(chain)
链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一 条或数条规则。当一个数据包到达一个链时，iptables就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据 该条规则所定义的方法处理该数据包；否则iptables将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables就会根据该链预先定 义的默认策略来处理数据包。

## 配置命令
iptables命令的使用情况如下
```bash
iptables [option] CHAIN_rule [-j target]
```
- –A, ––append 将规则添加到链中（最后）。
- –I, ––insert 将规则添加到给定位置的链中。
- –C, ––check 寻找符合链条要求的规则。
- –D, ––delete 从链中删除指定的规则。
- –F, ––flush 删除对应表的所有规则，慎重使用。
- –L, ––list 连锁显示所有规则。
- –v, ––verbose 使用列表选项时显示更多信息。
- -P, --policy 设置链的默认策略(policy)
- -N, --new 创建用户自定义链
- -X, --delete-chain 删除用户自定义链
- -E, --rename-chain 重命名用户自定义链

### 查看iptables规则
```bash
# 查看规则  默认情况下是filter 所以命令等于
# iptables -t filter -L
iptables -L  
[root@ops-2 ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy DROP)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

# 如上表示防火墙没有做任何的规则,接收任何的包
# 查看nat表的规则
iptables -t nat -L
[root@ops-2 ~]# iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
```
现在我们给他来一条数据,比如禁止10.0.6.35的数据包
```bash
# 当前ip是10.0.6.39 , 并且两台机器是可以互相ping通的
iptables -t filter -A INPUT -s "10.0.6.35" -j DROP
iptables -t filter -L
# Chain INPUT (policy ACCEPT)
# target     prot opt source               destination
# DROP       all  --  10.0.6.39            anywhere

# 再次去ping的时候就不通了
```

### 添加规则
```bash
# -A表示添加一条规则
# -i指定网卡
# -s指定数据包源地址
# -j指定动作选项
# 网卡eth0上来自172.18.0.3的数据包都丢弃
iptables -A INPUT -i eth0 -s 172.18.0.3 -j DROP
```

### 删除规则
```bash
# -D 按规则号或者规则内容进行匹配并删除
# -D  <链名> <规则号码  | 具体规则内容>
# 删除第三条规则
iptables -D INPUT 3
# 删除指定内容的数据
iptables -D INPUT -s 10.0.6.35 -j DROP
```

### 插入规则
```bash
# -I <链名> [规则号码]
# 插入规则是按照序号来插入的 默认情况下是插入到头部
# 这里有个问题,假设filter表INPUT链中的规则号码最大是2 当你根据序号 4插入的时候就会出现报错
# iptables: Index of insertion too big. 
# 插入的序号范围必须是当前已有序号的[1,最大值+1]
iptables -t filter -I INPUT 3 -s 10.0.6.35 -j DROP
```

### 替换规则
```bash
# -R <链名> <规则号> <具体规则内容>
# 规则号必须存在
iptables -t filter -R INPUT 3 -s 172.18.0.6 -j ACCEPT
```

### 默认策略
用于某个链的默认规则。当数据包没有被规则列表里的任何规则匹配到时，按此默认规则处理
- 默认策略DROP，需要逐条允许。一般服务器上都是这么设置的，仅开放部分端口或者白名单，其他全部拒绝。
- 默认策略ACCEPT，需要逐条拒绝。

> 正常情况下要先设置白名单,然后设置默认策略
> 对于高要求的设备来说,设置白名单的形式更加适用,但是白名单的要求就在于,需要同时设置OUPUT口和INPUT口
> 对于低要求的设备来说,设置黑名单就够了

如果防火墙之前没有任何配置，当上面的命令执行后，如果你当前是使用ssh连接的服务器，你会发现你被踢出来了
`远程设置时不要一开始就写这几句默认策略，会把自己踢出服务器，应该最后设定。`

这里是 `大写的P`
```bash
iptables -P INPUT DROP 
iptables -P FORWARD DROP 
iptables -P OUTPUT DROP
```



### 清除规则
不影响 -P 设置的默认规则
`当只清除对应表链中的规则的时候 -P 设置了 DROP 后，使用 -F 一定要小心。因为相当于把所有ACCEPT的都清除了，会导致ssh连接不上`
如果不写链名，默认清空某表里所有链里的所有规则。
```bash
ip tables -F
# 只清除对应表链中的规则 
iptables -t filter -F INPUT
```

### 保存规则
用iptables命令写的规则都是立即生效的，但只是临时的。
```bash
iptables-save # 可以查看全部的规则
```
centos7上面默认使用的是firewalld,如果需要使用iptables的话
```bash
systemctl disable firewalld
yum install iptables-services
systemctl enable iptables
service iptables save
```


## iptables匹配选项

### 匹配协议类型
用于指定规则的协议,如tcp,udp,icmp等,可以使用all来指定所有协议,这里是`小写的p`,默认值是all
协议的映射关系是`/etc/protocols`
```bash
# 禁止目标IP的ping ping的协议是icmp协议
iptables -A INPUT -s 172.18.0.3 -p icmp  -j DROP
```
这里是禁止了来源于`172.18.0.3`这台机器的icmp协议, 所以这台机器是ping不通当前机器的,但是当前机器是可以发送icmp包到达`172.18.0.3`这台机器的(原因是因为OUTPUT链并没有禁止)

### 源地址
源地址可以允许单个地址,一个网段,取反等等操作
```bash
-s 192.168.0.1 匹配来自 192.168.0.1 的数据包
-s 192.168.1.0/24 匹配来自 192.168.1.0/24 网络的数据包
-s 192.168.0.0/16 匹配来自 192.168.0.0/16 网络的数据包
! -s 192.168.1.101除这个IP外
```

### 目的地址
用于指定目的地址,参数与`-s`相同
```bash
-d 202.106.0.20 匹配去往 202.106.0.20 的数据包
-d 202.106.0.0/16 匹配去往 202.106.0.0/16 网络的数据包
-d www.abc.com 匹配去往域名 www.abc.com 的数据包
```

### 执行目标
用于指定target。可能的值是ACCEPT, DROP, QUEUE, RETURN.

### 指定网卡输入
指定网卡
```bash
-i eth0指定了要处理经由eth0进入的数据包。
如果不指定-i参数，那么将处理进入所有接口的数据包。
如果出现! -i eth0，那么将处理所有经由eth0以外的接口进入的数据包。
如果出现-i eth +，那么将处理所有经由eth开头的接口进入的数据包。
```

### 指定网卡输出
用于指定数据包由哪个接口输出，这些数据包即将进入FORWARD, OUTPUT, POSTROUTING链。
```bash
如果不指定-o选项，那么系统上的所有接口都可以作为输出接口。
如果出现! -o eth0，那么将从eth0以外的接口输出。
如果出现-i eth +，那么将仅从eth开头的接口输出。
```

### 指定源端口 
只针对tcp 或者 udp协议
```bash
--sport 1000       匹配源端口是 1000 的数据包
--sport 1000:3000  匹配源端口是 1000-3000 的数据包（含1000、3000）
--sport :3000      匹配源端口是 3000 以下的数据包（含 3000）
--sport 1000:      匹配源端口是 1000 以上的数据包（含 1000）
```

### 指定目的端口 
只针对tcp 或者 udp
```bash
# 允许外部数据访问本地服务器80端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

## iptables动作选项

### ACCEPT
允许数据包通过,一般白名单的时候用
```bash
iptables -A INPUT -s 172.18.0.3 -j ACCEPT
```

### DROP
阻止数据包,直接丢弃
```bash
iptables -A INPUT -s 172.18.0.3 -j DROP
```


### DNAT
目的地址转换,将数据包转换到内部服务器,支持单IP ,也支持IP地址池
```bash
-j DNAT --to-destination 内网服务ip:端口
```
```bash
# 把流入eth0 要访问 TCP/80 的数据包目的地址转发到 172.18.0.3
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 172.18.0.3

# 把从 eth0 进来的要访问 TCP/81 的数据包目的地址改为 172.18.0.3:80
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 81 -j DNAT --to 172.18.0.3:80
    
# 把从 eth0 进来的要访问 TCP/80 的数据包目的地址改为地址池 172.18.0.3-172.18.0.10
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 172.18.0.3-172.18.0.10
```

### SNAT
源地址转换,共享内部主机上网
```bash
# 将 172.18.0.3 的原地址修改为 172.18.0.2
iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j SNAT  --to 172.18.0.2 

iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j MASQUERADE 

# 将内网 172.18.0.1/24 的原地址修改为 1.1.1.1
iptables -t nat -A POSTROUTING -d 172.18.0.1/24  -j SNAT --to 1.1.1.1

# 同上，只不过修改成一个地址池里的 IP
iptables -t nat -A POSTROUTING -d 172.18.0.1/24  -j SNAT --to 1.1.1.1-1.1.1.10
```


## 模块选项
`-m` 参数支持模块选项

```bash
state: 按包状态匹配
mac: 按来源 MAC 匹配
limit: 按包速率匹配
multiport: 多端口匹配
iprange: 按IP范围匹配
string 按字符串限定
```

### 按包状态匹配
```bash
-m state --state 状态
NEW：新建连接请求的数据包，且该数据包没有和任何已有连接相关联。
ESTABLISHED：该连接是某NEW状态连接的回包，也就是完成了连接的双向关联。
RELATED：衍生态，与 conntrack 关联。简而言之，A连接已经是ESTABLISHED，而B连接如果与A连接相关，那么B连接就是RELATED。
INVALID：匹配那些无法识别或没有任何状态的数据包。这可能是由于系统内存不足或收到不属于任何已知连接的ICMP错误消息，也就是垃圾包，一般情况下我们都会DROP此类状态的包。
```
拒绝新包,允许已经连接或已经有连接相关的数据包
```bash
iptables -A INPUT -p tcp -m state --state NEW -j DROP
iptables -A INPUT -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT
```

### 按包状态匹配新
```bash
-m conntrack --ctstate 状态
NEW：新建连接请求的数据包，且该数据包没有和任何已有连接相关联。
ESTABLISHED：该连接是某NEW状态连接的回包，也就是完成了连接的双向关联。
RELATED：衍生态，与conntrack关联。简而言之，A连接已经是ESTABLISHED，而B连接如果与A连接相关，那么B连接就是RELATED。
INVALID：匹配那些无法识别或没有任何状态的数据包。这可能是由于系统内存不足或收到不属于任何已知连接的ICMP错误消息，也就是垃圾包，一般情况下我们都会DROP此类状态的包。
UNTRACKED:这是一种特殊状态，或者说并不是状态。它是管理员在raw表中，为连接设置NOTRACK规则后的状态。这样做，便于提高包过滤效率以及降低负载
```
FORWARD 链的请求如果目标是 docker0 所在的网段, 而且已经建立的连接或者和已建立连接相关那么接受请求
```bash
iptables -A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

### 匹配某个mac地址
```bash
iptables -A FORWARD -m --mac-source xx:xx:xx:xx:xx:xx -j DROP
```

### 限速度
```bash
-m limit --limit 匹配速率 [--burst 缓冲数量]
-m limit --limit 25/minute: 允许最多每分钟25个连接
-m limit --limit-burst 100: 当达到100个连接后，才启用上述25/minute限制


iptables -A FORWARD -d 192.168.0.1 -m limit --limit 50/s -j ACCEPT
iptables -A FORWARD -d 192.168.0.1 -j DROP
```
注意: limit仅仅是用一定的速率去匹配数据包,并非限制


## 防火墙实例

### 简单防火墙
- loop回环网卡流量
- 连接状态为RELATED,ESTABLISHED的连接
- 80,22,21端口

```bash
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED  -j ACCEPT
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED  -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```

### 黑名单
```bash
iptables -A INPUT -i eth0 -s 172.18.0.3 -j DROP
```

### 端口转发
```bash
# iptables -t nat -F
# 流入当前机器80端口tcp流量转发到172.18.0.3:80
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to 172.18.0.3:80

# 流出当前机器80端口tcp流量源地址修改为172.18.0.2
iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j SNAT --to 172.18.0.2 
# 或者使用下面的命令: 目标IP地址自动获取
# iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 172.18.0.3 -j MASQUERADE
```

## iptables命令合集
```bash
iptablse -t [TABLE] -L # 查看对应表中的规则

# -v 详细显示 -n 以数字形式显示 --line-number 打印序号,序号为添加的顺序
iptables -t [TABLE] -vnl  --line-number # 详细显示

# 网卡eth0上来自172.18.0.3的数据包都丢弃
iptables -A INPUT -i eth0 -s 172.18.0.3 -j DROP

# 删除第三条规则
iptables -D INPUT 3

# 删除指定内容的数据
iptables -D INPUT -s 10.0.6.35 -j DROP
```


## iptables防火墙规则的保存与恢复
iptables持久化保存,防火墙设置后, 系统重启就会消失,为了持久化防火墙规则,需要将防火墙保存到`rc.d`脚本下面
```bash
iptables-save把规则保存到文件中，再由目录rc.d下的脚本（/etc/rc.d/init.d/iptables）自动装载
使用命令iptables-save来保存规则。一般用
iptables-save > /etc/sysconfig/iptables
或者
service iptables save
它能把规则自动保存在/etc/sysconfig/iptables中。
当计算机启动时，rc.d下的脚本将用命令iptables-restore调用这个文件，从而就自动恢复了规则。
```

<font color='red'>注意这些操作之前,做好先做好测试,不然无法使用重启的方法来重置防火墙了.到时候就真的进不去了...</font>


## iptables规则执行顺序
iptables规则执行顺序通俗点将就是看line-number,从大到小过滤
```bash
[root@ops ~]# iptables -L --line-number
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     icmp --  10.0.6.35            anywhere
2    DROP       icmp --  10.0.6.35            anywhere
```
比如像这个,显示DROP 然后ACCEPT  最后结果就是 ACCEPT

```bash
[root@ops ~]# iptables -L --line-number
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    DROP       icmp --  10.0.6.35            anywhere
2    ACCEPT     icmp --  10.0.6.35            anywhere
3    DROP       icmp --  10.0.6.35            anywhere
```
像这个就是拒绝

```bash
[root@ops ~]# iptables -L --line-number
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  192.168.1.203        anywhere             tcp dpt:gt-proxy
2    ACCEPT     tcp  --  192.168.1.202        anywhere             tcp dpt:gt-proxy
3    ACCEPT     tcp  --  192.168.1.201        anywhere             tcp dpt:gt-proxy
4    DROP       tcp  --  anywhere             anywhere             tcp dpt:gt-proxy
```
通过iptables限制9889端口的访问（只允许192.168.1.201、192.168.1.202、192.168.1.203）,其他ip都禁止访问

## iptables常用规则

1.拒绝进入防火墙的所有ICMP协议数据包
```bash
iptables -I INPUT -p icmp -j REJECT # 头部添加
iptables -t filter -A INPUT -p icmp -j REJECT # 末尾添加
```
2.允许防火墙转发除ICMP协议以外的所有数据包
```bash
iptables -A FORWARD -p ! icmp -j ACCEPT
```
3.拒绝转发来自192.168.1.10主机的数据，允许转发来自192.168.0.0/24网段的数据
```bash
iptables -A FORWARD -s 192.168.1.11 -j REJECT
iptables -A FORWARD -s 192.168.0.0/24 -j ACCEPT
```


https://www.cnblogs.com/sunsky303/p/12327863.html
http://xstarcd.github.io/wiki/Linux/iptables_forward_internetshare.html
https://my.oschina.net/mojiewhy/blog/3039897
https://my.oschina.net/deanzhao/blog/3058904
https://www.linuxprobe.com/25-iptables-common-examples.html
https://blog.csdn.net/qq_17004327/article/details/88630194
https://blog.csdn.net/watermelonbig/article/details/80319766
https://blog.csdn.net/qq_17004327/article/details/88630194
https://www.cnblogs.com/52fhy/p/13195418.html