## Snmp是什么
简单网络管理协议（SNMP）是TCP/IP协议簇的一个应用层协议。在1988年被制定，并被Internet体系结构委员会（IAB）采纳作为一个短期的网络管理解决方案；由于SNMP的简单性，在Internet时代得到了蓬勃的发展，1992年发布了SNMPv2版本，以增强SNMPv1的安全性和功能。现在，已经有了SNMPv3版本。

## snmp干什么
 简单网络管理协议（SNMP：Simple Network Management Protocol）是由互联网工程任务组（IETF：Internet Engineering Task Force ）定义的一套网络管理协议。该协议基于简单网关监视协议（SGMP：Simple Gateway Monitor Protocol）。**利用SNMP，一个管理工作站可以远程管理所有支持这种协议的网络设备，包括监视网络状态、修改网络设备配置、接收网络事件警告等。** 虽然SNMP开始是面向基于IP的网络管理，但作为一个工业标准也被成功用于电话网络管理


## 介绍一下snmp
在使用snmp的工具前，我们要先了解一下snmp的结构。
一套完整的SNMP系统主要包括管理信息库（MIB）、管理信息结构（SMI）及SNMP报文协议。
在结构上又分为NMS和Agent
![](https://img-blog.csdn.net/20130912155414250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2hhbnpoaXpp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
https://blog.csdn.net/shanzhizi/article/details/11606767
https://www.ibm.com/developerworks/cn/linux/l-cn-snmp/index.html


NMS network-manager-system 网络管理系统
基于TCP/IP的网络管理包含两个部分:网络管理站(也叫管理进程，manager)和被管的网络单元(也叫被管设备)。被管的设备种类繁多，例如:路由器、X终端、终端服务器和打印机等。这些被管设备的共同点就是都运行TCP/IP协议。被管设备端和关联相关的软件叫做代理程序(agent)或代理进程。管理站一般都是带有色彩监视器的工作站，可以显示所有被管理设备的状态(例如连接是否掉线、各种连接上的流量状况等)。

MIB manager-infomation-base 管理信息库包含所有代理进程的所有可被查询和修改的参数
SMI Structure of Management Information 关于MIB的一套共用的结构和表示符号。叫做管理信息结构
SNMP Simple Network Management Protocol 简单网络管理协议 管理进程和代理进程之间的通信协议


## 协议
管理进程和代理进程之间的通信可以有两种方式，一种是管理进程向代理进程发出请求，**询问(get)**一个具体的参数值(例如询问你产生了多少个不可达的ICMP端口) **修改(set)** 按要求改变代理进程的参数值  另外由代理端向管理端**主动汇报某些内容(trap)**(例如:一个接口掉线了)


## 请求方法
1)get_request (询问) 从代理进程处提取一个或多个参数值
2)get_next_request (下一次询问) 从代理进程处提取一个或多个参数的下一个参数值
3)set_request (修改) 设置代理进程一个或多个参数值
4)get_response (获取请求响应)
5)trap 代理进程主动发出的报文，通知管理进程有某些事发生。

![](https://img-blog.csdn.net/20170420151804336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvajAwMzYy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


**这些操作的前4钟操作是简单的请求应答方式，而且在SNMP钟往往使用UDP协议，所以可能发生管理进程和代理进程之间数据包丢失的情况。因此一定要有超时和重传机制。**

管理进程发出的前面三种操作采用UDP的161端口，而trp走的是162端口

![](https://img-blog.csdn.net/20170420161159317)








## 在Linux下使用snmp
#### 安装
```bash
yum install net-snmp net-snmp-utils -y
# 查看是否安装完成
snmpd -v
snmpwalk -V
```
其中net-snmp是snmp软件
net-snmp-utils 是snmp的工具

#### 使用
开启snmp服务，snmpd是一个linux的服务，但是我们是通过使用snmpwalk来操作snmp这个服务的
```bash
systemctl start snmpd
```
常用的snmpwalk数
```bash
-h  # 显示帮助
-v  # 指定snmp的版本，1或者2或者3
-c  # 指定连接设备snmp密码。
-V  # 显示当前snmpwalk命令行版本
-r  # 指定重试次数，默认为0次
-t  # 指定每次请求的等待超时
-l  # 指定安全级别:noAuthNoPriv|authNoPriv|authPriv
-a  # 验证协议：MD5|SHA。只有-l指定为authNoPriv或authPriv时才需要
–A  # 验证字符串。只有-l指定为authNoPriv或authPriv时才需要
–x  # 加密协议：DES。只有-l指定为authPriv时才需要
–X  # 加密字符串。只有-l指定为authPriv时才需要。
```
常用方法
```bash
snmpwalk -v 2c -c public 10.1.1.1 .1.3.6.1.2.1.25.1   # 得到取得windows端的系统进程用户数等
# 其中 -v指定版本  -c 指定密码  10.1.1.1 指定地址(localhost表示本地)  .1.3.6.1.2.1.25.1表示oid 不同的oid表示了不同的操作
```

## 常用oid
http://www.ttlsa.com/monitor/snmp-oid/