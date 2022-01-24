---
title: question
date: 2021-06-16 14:22:18
categories:  [面试]
tags: [面试]
---


<!--more-->


# question

Linux下查看磁盘空间大小（容量判断）、索引号容量（文件数量判断） ?

```bash
# 查看磁盘空间大小
df -h
# 查看索引号容量
df -i
```


Linux 下如何分区大于2T的磁盘 ？（写个命令名）
```bash
小于2T fdisk
大于2T parted
```

Vi中的y0 ，d$ ，gg ，G ，dd ，5yy，cw5，x 分别表示什么？

```bash
- y0 复制光标所在行到最后一行结尾的所有内容
- d$ 删除当前所在光标到行结尾所有的内容
- gg 跳转到第一行
- G 跳转到最后一行
- dd 删除光标所在行
- 5yy 光标所在行及其下面4行所有的内容
- 应该是c5w 这里是删除5个单词并进入编辑模式  cw删除一个单词 CW删除一个单词(默认以空白为分隔符号)
- x 删除一个单词 不进入编辑模式
```


TCP三次握手及四次挥手的详细过程解释？

```
三次握手
1. 发送端发起连接信号 (SYN=1;seq=client_id) 
2. 服务端接收客户端发来的信号 返回确认信号 (SYN=1;seq=service_id;ack=client_id+1)
3. 客户端接收服务端发来的信号 此时连接已经建立，返回一个报文段 (SYN=1;seq=client_id+1;ack=service_id+1)

四次挥手
1. 客户端发出终止信号(FIN=1)
2. 服务端收到终止信号，请求确认 (ack)
3. 服务端发出终止信号(FIN)
4. 客户端接收信号，发出确认(ack)
5. 服务端接收确认信息，关闭连接
6. 客户端发出确认信息后，等待一段时间后，关闭连接
```


Linux下如何创建一块网卡的网络配置？（假设不管mac，设备名为eth0）

```
cd /etc/sysconfig/network-scripts/
cp ifcfg-eth0 ifcfg-backup

echo << EOF > ifcfg-backup
IPADDR=10.0.5.158
NETMASK=255.255.240.0
GATEWAY=10.0.0.130
EOF

sed -i "/^ONBOOT/s/no/yes/"  ifcfg-backup
sed -i "/^BOOTPROTO/s/dhcp/static/" ifcfg-backup
sed -i "/^DEVICE/s/eth0/backup" ifcfg-backup
sed -i "/^NAME/s/eth0/backup" ifcfg-backup
```


linux作为NAT网关需要进行哪些配置？

做一个DNAT和SNAT的

DNAT 是目的端口转发， 外网通内网
SNAT 是源端口钻发， 内网通外网



```bash
echo 1 > /proc/sys/net/ipv4/ip_forward 
iptables -t nat -i eth0 -A PREROUTING -D 外网 -J DNAT --to 内网
iptables -t nat -o eth1 -A POSTROUTING -s 内网 -J SNAT --to 外网
```






交换机端口动态汇聚和静态汇聚的优缺点对比

```bash
动态汇聚与静态汇聚主要依赖的协议是lacp 链路聚合协议
链路聚合协议的主要作用是 将多条物理链路合并成一组逻辑链路，作用是增加带宽量

静态链路的缺点，端口的选中/非选中状态由人工建立，不能根据对端端口的状态调整当前端口的选中/非选中状态，优点是不受网络的限制，稳定
动态链路的缺点，初始阶段需要人工建立动态汇聚，建立完成之后，当前端口可以根据对端端口的状态，自动协商端口状态

当前公司配置主要使用动态汇聚的形式，这样后面扩上联口的时候，比较方便，划一个新的上联口到汇聚口就行了
```




使用一行命令统计系统当前的网络状态为TIME_WAIT的有多少条？
```bash
netstat -n | grep TIME_WAIT | wc -l
```

以下面一段文本为例，分别完成(可以使用bash,sed,awk)：
```bash
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
```


打印出组为0的行内容（符合结果有二条）

```bash
cat xx | awk -F ":" '{if ($4 == '0'){ print $0}}'
```



修改sync用户的命令为/sbin/nologin
```bash
# 方法一
sed -i "/^sync/s/\/bin\/sync/\/bin\/nologin/" xx
# 方法二
sed -i "/^sync/s#/bin/sync#/sbin/nologin#"  xx
```


在daemon用户后面插入一行
`mysql:x:500:500:mysql:/home/mysql:/bin/nologin`

```bash
# 方法一
echo mysql:x:500:500:mysql:/home/mysql:/bin/nologin | xargs -I {} sed -i "/^daemon/a{}" xx
# 方法二
sed -i "/^daemon/a mysql:x:500:500:mysql:/home/mysql:/bin/nologin" xx
```

思考题（没有正确答案，只要思路）
设计一个有100台左右服务器网络架构，有一个总计100G带宽的公网出口，需要考虑高可用和负载均衡。（交换机型号和使用数量不限，内部互联接口类型不限）
pass


罗列交换机IRF的优缺点。

```bash
优点
1. 高扩展性，多台交换机堆叠，可以做到端口互备 扩展带宽的作用
2. 统一管理，主要在主机上进行管理就可以了

缺点 
1. 配置麻烦，需要重启，步骤要按照要求来，很容易出现错误
2. 交换机厂商必须相同
```


描述动态路由和静态路由的区别。

```bash

静态路由 : 路由表固定
优点: 稳定简单效率高
确定: 不够灵活，不能适应网络的变化而变化
场景： 规模不大的场景

动态路由: 路由表不固定，网络间的路由器协商完成
优点： 灵活，适应网络变化
缺点： 受网络影响大，占带宽
场景  复杂网络 

```



数据中心内部网络需要开放一个对公网开放的服务，有哪些方法来实现？
```bash
1. 集群本地做NAT地址转发
2. frp内网穿透
3. 集群上分做层nginx代理转发
```


需要在一台服务器上对异常大流量进行定位分析，有哪些方法？
```bash
1. 看监控图 -> 定位节点 -> 定位机器  -> 定位程序 -> 定位日志
2. 定位程序的工具 iftop nethogs iptraf-ng

netstat -n | awk '/^tcp/ {++s[$NF]}{for (a in s ){print a s[a]}}'
```
