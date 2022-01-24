---
title: nginx基础
categories:
  - nginx
tags:
  - nginx
abbrlink: '21215442'
date: 2020-02-03 14:27:43
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [nginx](#nginx)
  - [nginx安装](#nginx安装)
  - [nginx常用命令](#nginx常用命令)
  - [nginx配置文件](#nginx配置文件)
  - [nginx文件配置](#nginx文件配置)
    - [location模块配置](#location模块配置)
  - [配置实例](#配置实例)
  - [nginx错误](#nginx错误)

<!-- /code_chunk_output -->

<!-- more -->

# nginx

## nginx安装
- 直接安装
```bash
yum install nginx 
# 查看版本
nginx -v
```
- 编译安装
安装gcc、make、wget、g++这些软件
0. 创建一个临时目录下载安装所需要的内容
```bash
cd /tmp
mkdir gen_nginx 
```
1. 下载安装openssl,主要用于ssl模块加密，支持htps 
```bash
wget https://www.openssl.org/source/openssl-1.0.2s.tar.gz
```
2. 下载pcre来实现对地址重定向，地址重写功能和localtion指令以及正则表达式的支持
```bash
wget https://ftp.pcre.org/pub/pcre/pcre-8.43.tar.gz
```
3. 下载zlib gzip压缩模块
```bash
wget https://zlib.net/zlib-1.2.11.tar.gz 
```
4. 下载nginx
```bash
wget http://nginx.org/download/nginx-1.17.1.tar.gz
```
5. 解压所有的文件
```bash
ls *.tar.gz | xargs -n1 tar xzvf 
```
6. 使用configure编译安装
具体的编译内容可以自己决定
```bash
./configure \
   --with-openssl=../openssl-1.0.2s \
   --with-pcre=../pcre-8.43 \
   --with-zlib=../zlib-1.2.11 \
   --with-pcre-jit --user=admin \
   --prefix=/home/admin/nginx \
   --with-http_ssl_module \
   --with-http_v2_module
```
7. 编译安装
```bash
make && make install
```
8. 分配权限
```bash
sudo chown root nginx 
sudo chmod u+s nginx
```

9. 查找安装路径
源码编译的时候，我们指定了一个prefix的文件，也就是nginx安装的目标目录 `/home/admin/nginx` 下面
![2020-05-12-13-49-59](http://noback.upyun.com/2020-05-12-13-49-59.png)

可以使用软连接把nginx指令映射到`/usr/bin/`下面
```bash
ln -s /home/admin/nginx/sbin/nginx /usr/bin/nginx
```


- 正向代理
局域网通过代理服务器访问Internet上的内容，这个过程叫做正向代理
- 反向代理
客户端发送请求到反向代理服务器，由反向代理服务器选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP的地址 
- 负载均衡
通过反向代理服务器，将多个请求平均分配到目标服务器上
- 动静分离
动态资源与动态资源分开放，由反向代理服务器来根据请求内容，将请求转发到指定目标服务器


## nginx常用命令
首先要进入到Nginx的目录下,如果是编译安装的话，nginx的目录应该在/usr/local/sbin/nginx中，如果是直接安装下载的话，nginx的目录应该在/usr/sbin/nginx

```bash
# 查看Nginx进程
ps -ef | grep nginx
# 查看nginx的版本号
./nginx -v
# 强制关闭Nginx
./nginx -s stop
# 退出nginx
./nginx -s quit
# 启动nginx
./nginx
# 重新加载nginx
./nginx -s reload 
```

## nginx配置文件
如果是直接安装，则配置文件在/etc/nginx/nginx.conf，如果是编译安装，则配置文件在/usr/local/nginx中
查找nginx配置文件
```bash
find / | grep nginx.conf  
```

nginx配置文件的组成主要分成3大块内容
- 第一部分: 全局模块
```bash
user nginx; # 用户 用户组 ; 一般只有类unix系统有,windows不需要
worker_processes auto; # 工作进程，一般配置为cpu核心数的2倍，值越大，可以支持的并发处理量越多
error_log /var/log/nginx/error.log; # 错误日志的配置路径
pid /run/nginx.pid; # 进程ID存放路径
include /usr/share/nginx/modules/*.conf; 
```
- 第二部分: events模块
主要影响nginx服务器与用户的网络连接
```bash 
events {
    # 使用epoll的I/O 模型；epoll 使用于Linux内核2.6版本及以后的系统
    use epoll;
    # 每一个工作进程的最大连接数量； 理论上而言每一台nginx服务器的最大连接数为： worker_processes*worker_connections
    worker_connections  1024;

    # 超时时间
    keepalive_timeout 60

    # 客户端请求头部的缓冲区大小，客户端请求一般会小于一页； 可以根据你的系统的分页大小来设定， 命令 getconf PAGESIZE 可以获得当前系统的分页大小（一般4K）
    client_header_buffer_size 4k;

    # 为打开的文件指定缓存，默认是不启用； max指定缓存数量，建议和打开文件数一致；inactive是指经过这个时间后还没有被请求过则清除该文件的缓存。
    open_file_cache max=65535 inactive=60s;

    # 多久会检查一次缓存的有效信息
    open_file_cache_valid 80s;

    # 如果在指定的参数open_file_cache的属性inactive设置的值之内，没有被访问这么多次（open_file_cache_min_uses），则清除缓存
    # 则这里指的是 60s内都没有被访问过一次则清除 的意思
    open_file_cache_min_uses 1;
}
```

- 第三部分1: http模块
http全局配置的指令包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单链请求数上限等。
```bash
http {
    # 日志发送
    # 日志格式  其中main是日志文件的名字
    # --------------------------------------------------------------------------   #
    # 日志格式设置：
    #   $remote_addr、$http_x_forwarded_for     可以获得客户端ip地址
    #   $remote_user                            可以获得客户端用户名
    #   $time_local                             记录访问的时区以及时间
    #   $request                                请求的url与http协议
    #   $status                                 响应状态成功为200
    #   $body_bytes_sent                        发送给客户端主体内容大小
    #   $http_referer                           记录从哪个页面过来的请求
    #   $http_user_agent                        客户端浏览器信息
    #
    #   注意事项：
    #           通常web服务器(我们的tomcat)放在反向代理(nginx)的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。
    #           反向代理服务器(nginx)在转发请求的http头信息中，可以增加$http_x_forwarded_for信息，记录原有客户端的IP地址和原来客户端的请求的服务器地址。
    # --------------------------------------------------------------------------   #
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # 指定日志文件存储地址
    access_log  /var/log/nginx/access.log  main;

    # sendfile 指定 nginx 是否调用sendfile 函数（零拷贝 方式）来输出文件；
    # 对于一般常见应用，必须设为on。
    # 如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    # 负载均衡 START>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    # upstream 指令定义的节点可以被proxy_pass指令引用；二者结合用来反向代理+负载均衡配置
    # 【内置策略】：轮询、加权轮询、ip_hash、最少连接 默认编译进了nginx
    # 【扩展策略】：fair、通用hash、一致性hash 默认没有编译进nginx
    #-----------------------------------------------------------------------------------------------#
    # 【1】默认是轮询；如果后端服务器down掉，能自动剔除。
    #   upstream bakend {
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #        server 192.168.75.134:8080;
    #    }
    #
    #【2】权重轮询(加权轮询)：这样配置后，如果总共请求了3次，则前面两次请求到130，后面一次请求到132
    #   upstream bakend {
    #        server 192.168.75.130:8080 weight=2;
    #        server 192.168.75.132:8080 weight=1;
    #   }
    #
    #【3】ip_hash：这种配置会使得每个请求按访问者的ip的hash结果分配，这样每个访客固定访问一个后端服务器，这样也可以解决session的问题。
    #   upstream bakend {
    #        ip_hash;
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #   }
    #
    #【4】最少连接：将请求分配给连接数最少的服务器。Nginx会统计哪些服务器的连接数最少。
    #   upstream bakend {
    #        least_conn;
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #   }
    #
    #
    #【5】fair策略(需要安装nginx的第三方模块fair)：按后端服务器的响应时间来分配请求，响应时间短的优先分配。
    #   upstream bakend {
    #        fair;
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #   }
    #
    #【6】url_hash策略（也是第三方策略）：按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
    #               在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method指定hash算法
    #   upstream bakend {
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #        hash $request_uri;
    #        hash_method crc32;
    #   }
    #
    #【7】其他设置，主要是设备的状态设置
    #   upstream bakend{
    #       ip_hash;
    #       server 127.0.0.1:9090 down;   # down 表示该机器处于下线状态不可用
    #       server 127.0.0.1:8080 weight=2;
    #       server 127.0.0.1:6060;
    #       
    #       # max_fails 默认为1； 最大请求失败的次数，结合fail_timeout使用；
    #       # 以下配置表示 192.168.0.100:8080在处理请求失败3次后，将在15s内不会受到任何请求了
    #       # fail_timeout 默认为10秒。某台Server达到max_fails次失败请求后，在fail_timeout期间内，nginx会认为这台Server暂时不可用，不会将请求分配给它。
    #       server 192.168.0.100:8080 weight=2 max_fails=3 fail_timeout=15;
    #       server 192.168.0.101:8080 weight=3;
    #       server 192.168.0.102:8080 weight=1;
    #       # 限制分配给某台Server处理的最大连接数量，超过这个数量，将不会分配新的连接给它。默认为0，表示不限制。注意：1.5.9之后的版本才有这个配置
    #       server 192.168.0.103:8080 max_conns=1000;
    #       server 127.0.0.1:7070 backup;  # 备份机；其他机器都不可用时，这台机器就上场了
    #       server example.com my_dns_resolve;  # 指定域名解析器；my_dns_resolve需要在http节点配置resolver节点如：resolver 10.0.0.1;
    #   }
    #
    #
    #
    #负载均衡 END<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
    #-------------------------------------------------------------------------------------------#

    #设定mime类型,类型由mime.types文件定义；  可以 cat nginx/conf/mime.types  查看下支持哪些类型
    include             /etc/nginx/mime.types;
    # 默认mime类型；  application/octet-stream 指的是原始二进制流
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf; 
```

- 第三部分2: http-server模块
```bash
# server模块 
server {
        listen       80 default_server; # 监听端口
        listen       [::]:80 default_server;
        server_name  localhost
        root         /usr/share/nginx/html; 

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        include /etc/nginx/default.d/*.conf;

        location / {

        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    } 
```
```bash
# 单文件 nginx.conf

#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       8888;  # 监听端口
        server_name  localhost; # 监听的ip  加起来就是监听本地8888的变化 127.0.0.1:8888 == 公网IP:8888
        location / {  # 根目录，不设置目录
            root   html;  # html静态文件，存放html文件的地方，nginx安装之后就会产生这么一个文件夹，与conf同级
            index  xx.html index.htm; # 匹配对应的html文件

        }
    }

}

```


## nginx文件配置
如果是直接安装的nginx，则nginx所安装的目录文件如下
```bash
root:/etc/nginx# ls -l
total 72
drwxr-xr-x. 2 root root   24 Feb  3 17:14 conf.d   # http-server配置
drwxr-xr-x. 2 root root    6 Oct  3 13:15 default.d 
-rw-r--r--. 1 root root 1077 Oct  3 13:15 fastcgi.conf
-rw-r--r--. 1 root root 1077 Oct  3 13:15 fastcgi.conf.default
-rw-r--r--. 1 root root 1007 Oct  3 13:15 fastcgi_params
-rw-r--r--. 1 root root 1007 Oct  3 13:15 fastcgi_params.default
-rw-r--r--. 1 root root 2837 Oct  3 13:15 koi-utf
-rw-r--r--. 1 root root 2223 Oct  3 13:15 koi-win
-rw-r--r--. 1 root root 5231 Oct  3 13:15 mime.types
-rw-r--r--. 1 root root 5231 Oct  3 13:15 mime.types.default
-rw-r--r--. 1 root root 2340 Feb  3 17:40 nginx.conf    # nginx配置
-rw-r--r--. 1 root root 2698 Feb  3 17:03 nginx.conf.default # 
-rw-r--r--. 1 root root  636 Oct  3 13:15 scgi_params
-rw-r--r--. 1 root root  636 Oct  3 13:15 scgi_params.default
-rw-r--r--. 1 root root  664 Oct  3 13:15 uwsgi_params
-rw-r--r--. 1 root root  664 Oct  3 13:15 uwsgi_params.default
-rw-r--r--. 1 root root 3610 Oct  3 13:15 win-utf 
```
在 nginx.conf配置中有这么一段话，其中表明了几个文件夹的作用
/etc/nginx/default.d/ 主要存放location
/etc/nginx/conf.d/ 主要存放server
否则会报错 <font color='red'>nginx: [emerg] "user" directive is not allowed here in /etc/nginx/conf.d/nginx.conf:5</font>
```bash
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information
    include /etc/nginx/conf.d/*.conf;   # 存放server

    server {
        listen       80;
        server_name  10.0.5.233;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf; # 存放location

        location / {
           root html;
           proxy_pass http://127.0.0.1:8001;
           index index.html index.htm;
        }
    }
```
--------------------
在/etc/nginx/nginx.conf中存放了一些初始的配置，可以选择在在这个文件中直接添加server来配置nginx，也可以将自己的server单独写到conf.d/文件夹中去，或者写到nginx.conf.default中去


### location模块配置
配置内容如下
```bash
location [ = | ~ | ~* | ^~ ]  uri {

}
```
= 用于不需要正则表达式的uri 要求严格匹配，如果匹配成功，就往下搜索并处理请求
~ 用于需要正则表达式的uri 区分大小写
~* 用于需要正则表达式的uri 不区分大小写
^~ 用于不需要正则表达式的uri，要求Nginx服务器找到标识uri和请求字符串匹配度最高的location后，立即使用此location处理请求，而不在使用location块中的正则uri和请求字符串做匹配


## 配置实例
![](https://img2018.cnblogs.com/blog/1381066/201906/1381066-20190627141117429-435885754.png)


## nginx错误
```bash
nginx bind() to 0.0.0.0:**** failed (13: Permission denied)
```
这是因为防火墙没有关闭或者selinux没有关闭
```bash
systemct  stop firewalld
setenforce 0
sed -in -r '/^.*=Enforce/s#Enforce#disabled#g' /etc/selinux/config
```


