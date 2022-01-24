---
title: keepalive+haproxy负载均衡
categories:
  - linux
tags:
  - keepalive
  - haproxy
  - linux
abbrlink: 14c89e63
date: 2020-01-07 17:45:56
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [keepalive](#keepalive)
  - [haproxy负载均衡](#haproxy负载均衡)
  - [单台haproxy+两台RS服务器](#单台haproxy两台rs服务器)
  - [keepalive+haproxy](#keepalivehaproxy)

<!-- /code_chunk_output -->

<!-- more -->
# keepalive
## haproxy负载均衡
```bash
# yum安装
yum install haproxy -y # centos7默认版本是1.5
# 查看包信息
rpm -qi haproxy
# 查看包所创建的目录
rpm -ql haproxy
```

------

---->主要目录
主程序：/usr/sbin/haproxy
配置文件：/etc/haproxy/haproxy.cfg
启动服务：systemctl start haproxy
停止服务：systemctl stop haproxy   
开机启动：systemctl enable haproxy

---->配置文件结构

haproxy配置文件可分为全局配置（globalsettings）和 代理配置（proxies），而代理段配置包含defaults、frontend、backend、listen。

global settings：#全局参数配置，主要用于定义haproxy进程管理安全及性能相关的参数
defaults \< name \>：#默认配置参数，下面的段继承该配置，名称是可选的,这配置默认配置参数可由下一个"defaults"所重新设定
frontend \< name \>：#前端配置，定义一系列监听的套接字，这些套接字可接受客户端请求并与之建立连接，可以监听多个端口。
backend \< name \>：#后端配置，定义后台服务器，前端代理服务器将会把客户端的请求调度至这些服务器，类似nginx中的upstream
listen \< name \>：#定义一组前端和后端的完整代理，可理解为frontend+backend，通常用于tcp流量代理

---->配置参数可支持的时间单位
- us : 微秒. 1 microsecond = 1/1000000 s
- ms : 毫秒. 1 millisecond = 1/1000 s
- s  : 秒  1s = 1000ms
- m  : 分 1m = 60s = 60000ms
- h  : 小时   1h = 60m = 3600s = 3600000ms
- d  : 天    1d = 24h = 1440m = 86400s = 86400000ms

--------


haproxy配置文件

---------

**全局配置参数(global)**
```bash
chroot: #修改haproxy的工作目录至指定的目录，并在放弃权限之前执行chroot（）操作，可以提升haproxy的安全级别，不过需要注意的是确保指定的目录为空目录且任何用户均不能有写权限;
daemon: #让haproxy以守护进程的方式工作于后台，其等同于"-D"选项的功能，当然，也可以在命令行中以"-db"选项将其禁用;
gid: #以指定的GID运行haproxy，建议使用专用于运行haproxy的GID，以避免因权限带来的风险;
group: #同gid，不过这里为指定的组名;
uid: #已指定的UID身份运行haproxy进程;
user: #同uid，但这里使用的为用户名;
log: #定义全局的syslog服务器，最多可以定义两个;
nbproc: #指定启动的haproxy进程个数，只能用于守护进程模式的haproxy；默认为止启动一个进程，鉴于调试困难等多方面的原因，一般只在但进程仅能打开少数文件描述符的场中才使用多进程模式;
pidfile: #pid文件的存放位置;
ulimit-n:  #设定每个进程所能够打开的最大文件描述符，默认情况下其会自动进行计算，因此不建议修改此选项;
node: #定义当前节点的名称，用于HA场景中多haproxy进程共享同一个IP地址时;
description: #当前实例的描述信息;
maxconn: #设定每个haproxy进程所接受的最大并发连接数，其等同于命令行选项"-n"，"ulimit-n"自动计算的结果正式参照从参数设定的;
maxpipes: #haproxy使用pipe完成基于内核的tcp报文重组，此选项用于设定每进程所允许使用的最大pipe个数，每个pipe会打开两个文件描述符，因此，"ulimit -n"自动计算的结果会根据需要调大此值，默认为maxcoon/4;
noepoll: #在linux系统上禁用epoll机制;
nokqueue: #在BSE系统上禁用kqueue机制;
nopoll: #禁用poll机制;
nosepoll: #在linux系统上禁用启发式epoll机制;
nosplice: #禁止在linux套接字上使用tcp重组，这会导致更多的recv/send调用，不过，在linux2.6.25-28系列的内核上，tcp重组功能有bug存在;
spread-checks<0..50,in percent>: #在haprorxy后端有着众多服务器的场景中，在紧缺是时间间隔后统一对中服务器进行健康状况检查可能会带来意外问题，此选项用于将检查的时间间隔长度上增加或减少一定的随机时长，为当前检查检测时间的%;
maxconnrate：#设置每个进程每秒种所能建立的最大连接数量，速率，一个连接里可以有多个会话，也可以没有会话
maxsessrate：#设置每个进程每秒种所能建立的最大会话数量
maxsslconn：#每进程支持SSL 的最大连接数量
tune.bufsize: #设定buffer的大小，同样的内存条件下，较小的值可以让haproxy有能力接受更多的并发连接，较大的值了可以让某些应用程序使用较大的cookie信息，强烈建议使用默认值;
tune.chksize: #设定检查缓冲区的大小，单位为字节，更大的值有助于在较大的页面中完成基于字符串或模式的文本查找，但也会占用更多的系统资源，不建议修改;
tune.maxaccept: #设定haproxy进程内核调度运行时一次性可以接受的连接的个数，较大的值可以带来较大的吞吐量。
tune.maxpollevents: #设定一次系统调用可以处理的事件最大数，默认值取决于OS,其至小于200时可介于带宽，但会略微增大网络延迟，但大于200时会降低延迟，但会稍稍增加网络带宽的占用;
tune.maxrewrite: #设定在首部重写或追加而预留的缓存空间，建议使用1024左右的大小，在需要更大的空间时，haproxy会自动增加其值;
tune.rcvbuf.client: #设定内核套接字中客户端接收缓存区的大小，单位为字节，强烈推荐使用默认值;
tune.rcvbuf.server: #设定内核套接字中服务器接收缓存区的大小，单位为字节，强烈推荐使用默认值;
tune.sndbuf.client: #设定内核套接字中客户端发送缓存区的大小，单位为字节，强烈推荐使用默认值;
tune.sndbuf.server: #设定内核套接字中服务器端发送缓存区的大小，单位为字节，强烈推荐使用默认值;
debug: #调试模式，输出启动信息到标准输出;
quiet: #安装模式，启动时无输出; 
```

**global默认配置**
```bash
# 全局配置
global
    log 127.0.0.1 local0         # 设置日志
    log 127.0.0.1 local1 notice
    maxconn 4000                 # 最大连接数
    chroot /usr/local/haproxy    # 安装目录
    user haproxy
    group haproxy
    daemon                       # 守护进程运行
    #nbproc 1                    # 进程数量，只能用于守护进程模式的haproxy；默认启动一个进程，一般只在单进程仅能打开少数文件描述符的场景中才使用多进程模式；
    pidfile /var/run/haproxy.pid # pid文件的存放位置
```


**defaults默认配置**
```bash
defaults
　　mode http   #默认负载均衡模式为http
　　log global    #日志定义
　　option httplog   #启用日志记录HTTP请求，默认不记录http log
　　option dontlognull   #不记录空日志
　　option httpclose   # 每次请求完毕后主动关闭http通道
　　option forwardfor  except 127.0.0.0/8 #插入x-forward标记，反向代理时候可以通过该字段获取客户端真实IP
   balance roundrobin # 负载均衡算法,轮询
　　retries 3   # 定义连接后端服务器的失败重连次数
　　timeout http-request 10s：#在客户端建立连接但不请求数据时，关闭客户端连接
   timeout queue 1m : #服务器的maxconn时，连接在队列中保持挂起状态而设置的超时时间，想客户端返回503错误
   timeout connect 10s： #定义haproxy将客户端请求转发至后端服务器所等待的超时时长
   timeout client 1m：#客户端非活动状态的超时时长
   timeout server 1m：#客户端与服务器端建立连接后，等待服务器端的超时时长
   timeout http-keep-alive 10s: #定义保持连接的超时时长
   timeout check           10s: #健康状态监测时的超时时间，过短会误判，过长资源消耗
   maxconn                 3000: #每个server最大的连接数 

   # 超时参数
   timeout http request ：#在客户端建立连接但不请求数据时，关闭客户端连接
   timeout queue ：#等待最大时长
   timeout connect： #定义haproxy将客户端请求转发至后端服务器所等待的超时时长
   timeout client：#客户端非活动状态的超时时长
   timeout server：#客户端与服务器端建立连接后，等待服务器端的超时时长，
   timeout http-keep-alive ：#定义保持连接的超时时长
   timeout check：#健康状态监测时的超时时间，过短会误判，过长资源消耗 
```

**listen默认配置**

```bash
# 监听页面配置
listen admin_stats  
    bind 0.0.0.0:50000           # 监听IP和端口，为了安全可以设置本机的局域网IP及端口；
    mode http
    option httplog               # 采用http日志格式  
    stats refresh 30s            # 统计页面自动刷新时间 
    stats enable                 # 启用状态统计报告
    stats uri /stats    # 状态管理页面，通过 ip+port/stats来访问  如10.0.5.92:9999/stats

    stats realm "LOGIN"  # 密码框上显示的文本
    stats auth admin:psadmin     # 统计页面用户名和密码设置  
    stats hide-version          # 隐藏统计页面上HAProxy的版本信息
    #errorfile 403 /usr/local/haproxy/examples/errorfiles/   #设置haproxy 错误页面 
    stats admin_stats if TRUE #如果认证通过就做管理功能，可以管理后端的服务器
```

**前端配置**
```bash
frontend http_main
    bind 0.0.0.0:80              # http请求的端口，会被转发到设置的ip及端口
    # bind :80 # 监听本机所有ip80端口
    # bind *:80
    # bind 192.168.12.1:8080,10.1.0.12:8090

    
    # 转发规则
    #acl url_yuming   path_beg www.yuming.com
    #use_backend server_yuming if url_yuming

    # 默认跳转项，当上面都没有匹配上，就转到backend的http_default上；
    default_backend http_default

    # 提升失败的时候的用户体验
    #errorfile 502 /usr/local/haproxy/examples/errorfiles/502.http
    #errorfile 503 /usr/local/haproxy/examples/errorfiles/503.http
    #errorfile 504 /usr/local/haproxy/examples/errorfiles/504.http
```


**后端配置**
```bash
backend http_default
    # 额外的一些设置，按需使用
    option forwardfor
    option forwardfor header Client-IP
    option http-server-close
    option httpclose

    # 负载均衡方式
    #source 根据请求源IP
    #static-rr 根据权重
    #leastconn 最少连接先处理;在有着较长时间会话的场景中推荐使用此算法，如LDAP、SQL等，其并不太适用于较短会话的应用层协议，如HTTP；此算法是动态的，
    #uri 根据请求的uri
    #url_param 根据请求的url参数
    #rdp-cookie 据据cookie(name)来锁定并哈希每一次请求
    #hdr(name) 根据HTTP请求头来锁定每一次HTTP请求
    #roundrobin 轮询方式
    balance roundrobin           # 负载均衡的方式,轮询方式

    # 设置健康检查页面
    #option httpchk GET /index.html

    #传递客户端真实IP
    option forwardfor header X-Forwarded-For

    # 需要转发的ip及端口
    # inter 2000 健康检查时间间隔2秒
    # rise 3 检测多少次才认为是正常的
    # fall 3 失败多少次才认为是不可用的
    # weight 30 权重
    server node1 192.168.1.101:8080 check inter 2000 rise 3 fall 3 weight 30
    server node2 192.168.1.101:8081 check inter 2000 rise 3 fall 3 weight 30
```


**常用参数**
bind 用于监听
```bash
bind [<address>]:<port_range> [,...] [param*] 
# 仅在frontend和listen区域使用
bind 0.0.0.0:80              # http请求的端口，会被转发到设置的ip及端口
    # bind :80 # 监听本机所有ip80端口
    # bind *:80
    # bind 192.168.12.1:8080,10.1.0.12:8090
```

---------





## 单台haproxy+两台RS服务器

配置
haproxy 服务器 10.0.5.93
RS 服务器1  10.0.5.90
RS 服务器2  10.0.5.91

初始化准备
如果对防火墙没有太大的要求，可以直接关闭防护墙 所有机器都关闭
```bash
# 关闭防火墙
systemctl stop firewalld
systemctl status firewalld

# 安装haproxy
yum install haproxy -y

# 配置
vi /etc/haproxy/haproxy.cfg
```

haproxy服务器 DS
```bash
# 基础配置
global
    log         127.0.0.1 local0
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    stats socket /var/lib/haproxy/stats

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    balance roundrobin                      
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000 

# 开启监听端口
listen Status                   # 名字随便用
  bind 10.0.5.93:9999           # 绑定监听端口
  mode http                     
  stats enable
  stats uri /stats
  stats realm "LOGIN"
  stats refresh 6s
  stats auth upyun:upyun123

# 端口转发
listen web_server               # 名字随便用
  mode tcp                  #采用7层模式
  bind *:7777                   # 转发端口 请求7777 -> RS:8888 上面  因此这里的*:7777 可做vip -> keepalive
  balance roundrobin         #负载均衡算法，这里是轮换 # 轮询算法
  #option httpchk GET /test.test #健康检测   # 健康检测
  server web1 10.0.5.91:8888 weight 3 check inter 500 fall 3
  server web2 10.0.5.90:8888 weight 2 check inter 500 fall 3
```

查看haproxy  监听端状态
![2020-01-10-15-46-07](http://img.noback.top/2020-01-10-15-46-07.png)
不断请求10.0.5.93:7777 会发现ip会在90 与91 之间不断改变，因为我们采用了轮询的机制，相应的图中web total也会不断增加
```bash 
while 1; do curl http://10.0.5.93:7777/|grep "< h2 >.*< /h2 >" ;sleep 1; done
```
over

##  keepalive+haproxy
就是在上面的单台haproxy机器上  添加一台haproxy机器，同时利用keepalived的特性，使用vip在两台haproxy机器上漂移，无论vip漂移到哪个地方，都有对应的haproxy机器去对下面的两台RS机器进行负载均衡

keepalive 提高 haproxy的稳定性
haproxy 提高 RS的稳定性

配置
DS1   keepalive + haproxy  10.0.5.93 
DS2   keepalive + haproxy  10.0.5.92

RS1   10.0.5.91 
RS2   10.0.5.90 

VIP   10.0.5.96
监控端口  10.0.5.96:9999/stats
VIP端口 10.0.5.96:8888 ---- >  10.0.5.91:8888

DS1 keepalive配置
```bash
# 声明一下身份和vip就醒了
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER
    interface eno1
    virtual_router_id 51
    priority 1000
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
DS2 keepalived配置
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
```
这时候你可以先测试一下keepalived是否成功，两边开起来，然后关掉DS-master机器，如果VIP漂移到DS-BACKUP上面则表示成功

DS1 haproxy配置   与  DS2 haproxy相同
```bash
global
    log         127.0.0.1 local0
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    stats socket /var/lib/haproxy/stats

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    balance roundrobin
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

# 监听端口
listen Status
  bind 10.0.5.96:9999
  mode http
  stats enable
  stats uri /stats
  stats realm "LOGIN"
  stats refresh 6s
  stats auth upyun:upyun123

# 这个80端口有什么用
listen web_server
  mode tcp                  #采用7层模式
  bind *:7777
  balance roundrobin         #负载均衡算法，这里是轮换
  #option httpchk GET /test.test #健康检测
  server web1 10.0.5.91:8888 weight 3 check inter 500 fall 3
  server web2 10.0.5.90:8888 weight 2 check inter 500 fall 3
```

RS机器将VIP写入回源地址
```bash 
ifconfig enp4s0:0 10.0.5.96 broadcast 10.0.5.96 netmask 255.255.240.0 up
```

另外，当我切断master-keepalived的是否，会有一定时间的延迟
![2020-01-10-17-44-01](http://img.noback.top/2020-01-10-17-44-01.png)