---
title: linux命令tcpdump以及wireshark
date: 2020-12-11 10:50:23
categories: [linux,]  
tags: [linux,linuxcc]
---


<!-- more -->
# linux命令tcpdump

## 优化建议

tcpdump因为支持多参数模式匹配,可以使用and进行连接
这里的建议是and与and之间 用`(()and())` 这样可以看得清楚,比如

这里表示使用tcpdump抓取 来自10.0.6.41主机的TCP连接中的只含SYN包
```bash
tcpdump '((tcp) and (src host 10.0.6.41) and (tcp[tcpflags] = tcp-syn)) '
```

> 学习

学习tcpdump 建议搭配hping3 一起使用, hping3可以自己定制数据包
测试如下
```bash
# 接收端
tcpdump '((tcp) and (src host 10.0.6.41) and (tcp[tcpflags] = tcp-syn))'
# 发送端
hping3 -I eth0  -S 10.0.6.55
```

> 延时

tcpdump没有设定延时关闭, 可以使用timeout来控制,比如
抓10秒的包
```bash
timeout 10 tcpdump tcp and src host 10.0.6.41
```

> 指定head头部包,减少抓下来的包

https://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html
https://www.cnblogs.com/chyingp/p/linux-command-tcpdump.html
https://cizixs.com/2015/03/12/tcpdump-introduction/



## wireshark
wireshark有linux的命令版本， 叫做tshark。
这次做了抓包的分析的实验，在量小的情况下 全量大概500M 一分钟只抓128head字节 也有将近300M大小的包，测试需要1个小时
实际抓取过程中 1个小时大概抓了31G的包，但只有一个错误，错误内容如下
```bash
Jan 8, 2021 @ 17:30:35.000	10.31.196.2	1eecb132-e1ff-406e-9788-5b92339284f8	GET	timeout
```
如果单纯的去分析，这可以看好久，所以先截取某一段时间的包，切分的工具使用的是`editcap`
```bash
editcap -A "2021-01-08 17:20:35.000" -B "2021-01-08 17:50:35.000" bod0_time60_128bytes_4.cap  -F pcap bond0.pcap
```
但这样貌似是因为包太大了，会报错，所以采用按秒来分包
```bash
# 每10分钟一个包
editcap -i 600 bod0_time60_128bytes_4.cap bond0/
# 再每分钟分，往后推 推到时间点
editcap -i 60 _00004_20210109012317 per
```
也可以按照包的大小来分
```bash
# 100个包 这里太大了也不好用
editcap -c 100 bod0_time60_128bytes_4.cap out.pcap
```

写一个脚本 用来分析子包中出现的内容
```bash
# 匹配http请求中uri有 1eecb132-e1ff-406e-9788-5b92339284f8 的内容
find . -type f -size +500M  | cut -c 3- | while read line;do
        tshark -r $line  -Y 'http.request.uri matches "1eecb132-e1ff-406e-9788-5b92339284f8"' > ./${line:1:5}.log
done
```
匹配到以后 再把这个小文件传到电脑上用wireshark分析

wireshark 时间更新
https://www.cnblogs.com/LiuYanYGZ/p/13039202.html


## 参考链接
http://codeshold.me/2017/08/tcpdump_tshark_note.html