---
title: nginx配置实例
categories:
  - nginx
tags: 
  - nginx
abbrlink: 12ec35da
date: 2020-02-03 14:03:46
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [nginx配置实例](#nginx配置实例)
  - [nginx配置实例--反向代理1](#nginx配置实例-反向代理1)
  - [nginx配置--反向代理2](#nginx配置-反向代理2)
  - [nginx配置--负载均衡](#nginx配置-负载均衡)
    - [策略：](#策略)
  - [nginx配置实例--动静分离](#nginx配置实例-动静分离)
  - [nginx配置实例--高可用集群](#nginx配置实例-高可用集群)
  - [nginx原理](#nginx原理)

<!-- /code_chunk_output -->


<!-- more -->



# nginx配置实例


## nginx配置实例--反向代理1
配置内容
1台Nginx代理服务器(10.0.5.233)、一个网页
效果: 客户机在浏览器中输入www.123.com 由nginx代理进行转发到我们自己的网页上
```bash
# 首先 在客户机端做域名与ip的映射
echo "10.0.5.255 www.123.com" >> /etc/hosts
# 配置nginx文件 /etc/nginx/nginx.conf
server {
        listen       80;
        server_name  10.0.5.233;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
           root html;
           proxy_pass http://127.0.0.1:8001;
           index index.html index.htm;
        }
    } 
# 上面是nginx服务器的配置，表示的是监听10.0.5.233:80 将收到的请求转发到 http://127.0.0.1:8001本地端口
# 当我们在浏览器输入www.123.com:80 --> 10.0.5.233:80 --> server127.0.0.1:8001
```
## nginx配置--反向代理2
配置内容
1台Nginx代理服务器(10.0.5.233)、2个网页
效果: 客户机在浏览器中输入www.123.com/web1/ 跳转到第一个web页面 ; 输入www.123.com/web2/ 跳转到第二个web页面
```bash
# web页面自行配置,分别是
# 127.0.0.1:8080/web1/ -> web1
# 127.0.0.1:8081/web2/ -> web2
# 配置nginx文件 /etc/nginx/nginx.conf
server {
        listen       80;
        server_name  10.0.5.233;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        # ～ 表示后面的为正则表达式
        location ~ /web1/ {
           proxy_pass http://127.0.0.1:8080;
        } 
        location ~ /web2/ {
            proxy_pass http://127.0.0.1:8081;
        }
    } 
```


## nginx配置--负载均衡
配置内容
1台Nginx代理服务器(10.0.5.233) 2个web服务器
```bash
upstream myserver {
    server 10.0.5.197:8001;
    server 10.0.5.198:8001;
} 
server {
        listen       80;
        server_name  10.0.5.233;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        # ～ 表示后面的为正则表达式
        location  / {
           proxy_pass http://myserver;
        } 
    } 
```
### 策略： 
1. 默认策略为轮询
每个请求按时间顺序逐一分配到后端服务器，如果后端服务器down掉，能自动剔除
2. 权重策略
weight代表权重，权重默认为1，权重越高被分配的用户越多
```bash
upstream myserver {
    server 10.0.5.197:8001 weight=10;
    server 10.0.5.198:8001 weight=20;
} 
```
3. ip_hash策略
每个请求按照按访问ip的hash结果分配，每个访客固定访问一个后端服务器，可以解决session的问题
```bash
upstream myserver {
    ip_hash;
    server 10.0.5.197:8001;
    server 10.0.5.198:8001;
}  
```
4. fair策略
科举服务器响应时间来分配，响应的越快则优先分配
```bash
upstream myserver {
    server 10.0.5.197:8001;
    server 10.0.5.198:8001;
    fair;
}  
```

## nginx配置实例--动静分离
动态请求与静态请求分开，用nginx处理静态页面，django处理动态页面
- 静态文件放在静态资源管理服务器上，动态资源放在另外的服务器上，有Nginx做转发
- 静态文件和动态资源一起发布在同一个服务器上，由nginx做分离

```bash
server {
        listen       80;
        server_name  10.0.5.233;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location /image/ {
           root /image/;  # 静态文件路径
           index index.html index.htm;
        }
        location /data/ {
            root /data/;  # 静态文件路径
            autoindex on; # 自动列出目录
        }
    }  
```

## nginx配置实例--高可用集群
keepalived+nginx
配置需求：
master机器(main nginx) slave机器(backup nginx)  两台web服务器做轮询负载均衡 、 一个虚拟IP+4个机器IP

nginx活性检测脚本
```bash
#!/bin/bash
# 检测nginx进程数量
status=`ps -C nginx --no-header | wc -l`
# 如果没有进程，则重启之后再次检测nginx数量,如果依旧没有则杀掉所有keepalived的进程
if [ $status -eq 0 ];then
    /usr/sbin/nginx
    sleep 2
    if [ `ps -C nginx --no-header | wc -l ` -eq 0 ]; then
        killall keepalived
    fi
fi
```

keepalived配置,根据主机与从机的不通，只需要修改几个参数即可
router_id 、 state 、interface 、 virtual_router_id 、priority
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
   router_id  nginx_master
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

# nginx活性检测
vrrp_script chk_http_port{
    script "/usr/local/src/nginx_check.sh"

    interval 2 # 检测脚本执行的间隔

    weight 2 # 权重
}

vrrp_instance VI_1 {
    state BACKUP   # DS-backup机器 BACKUP MASTER
    interface eno1 
    virtual_router_id 51
    priority 10  # 优先级别
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.5.196/20
    }
} 
```

nginx配置
```bash
upstream myserver {
    server 10.0.5.197:8001;
    server 10.0.5.198:8001;
} 
server {
        listen       80;
        server_name  10.0.5.196;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        # ～ 表示后面的为正则表达式
        location  / {
           proxy_pass http://myserver;
        } 
    }  
```
![2020-02-04-15-41-24](http://noback.upyun.com/2020-02-04-15-41-24.png)
当用户访问10.0.5.196:80端口的时候，由于196是vip 此时正在master主机上面，因此他会读取master主机上面的nginx配置文件，后续因为nginx的负载均衡(轮询的配置)，当你不断的访问10.0.5.196时，页面会在10.0.5.197:8001 与10.0.5.198:8001直接来回的跳动



## nginx原理
https://www.bilibili.com/video/av68136734?p=17
https://www.cnblogs.com/dongye95/p/11096785.html#_label0_0
