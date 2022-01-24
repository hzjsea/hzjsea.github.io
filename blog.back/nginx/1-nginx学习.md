## 什么是nginx

-.0.0


## 使用nginx
nginx的文档存放在http://nginx.org/ 中。
nginx的下载地址存放在http://nginx.org/en/download.html 中
Centos版的nginx下载地址http://nginx.org/en/linux_packages.html#RHEL-CentOS
其中nginx一共有三个不同的版本
```bash
# 最新的版本，但不一定稳定
Mainline version  
# 稳定的版本
Stable version
# 历史稳定版本
Legacy version
```

**参数**
其中CHANGES是当前版本所做的改变，pgp则是安全认证


#### 下载安装
准备yum-utils
```bash
sudo yum install yum-utils
```
在/etc/yum.repos.d/nginx.repo下添加repo
```bash
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
```
在默认情况下我们安装的是稳定版本的nginx，当然官方也这么希望我们使用稳定版本的，如果要切换到最新版，需要添加一条命令
```bash
# 添加最新版，但不一定是最稳定的版本
sudo yum-config-manager --enable nginx-mainline
```
下载nginx
```bash
sudo yum install nginx
```

## nginx的目录详解
因为
```bash
# 查看nginx的所有安装目录
rpm -ql nginx
```
分别为一下一些目录
```bash
/etc/logrotate.d/nginx     配置文件   Nginx日志轮转，用于logrotate服务的日志切割

/etc/nginx
/etc/nginx/conf.d          配置文件   Nginx主配置文件
/etc/nginx/conf.d/default.conf

/etc/nginx/fastcgi_params
/etc/nginx/scgi_params    配置文件  cgi配置文件 fastcgi配置
/etc/nginx/uwsgi_params

/etc/nginx/mime.types   设置http协议的Content-Type与扩展名对应关系

/etc/sysconfig/nginx
/etc/sysconfig/nginx-debug         配置文件  用于配置出系统守护进程管理器管理方式
/usr/lib/systemd/system/nginx-debug.service
/usr/lib/systemd/system/nginx.service


/usr/lib64/nginx        nginx模块目录
/usr/lib64/nginx/modules


/usr/sbin/nginx
/usr/sbin/nginx-debug   Nginx服务的启动管理的终端命令

/usr/share/doc/nginx-1.16.1
/usr/share/doc/nginx-1.16.1/COPYRIGHT Nginx的手册和帮助文件
/usr/share/man/man8/nginx.8.gz


/var/cache/nginx Nginx缓存目录
/var/log/nginx  Nginx日志目录

/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/modules
/etc/nginx/nginx.conf
/etc/nginx/win-utf
/usr/libexec/initscripts/legacy-actions/nginx
/usr/libexec/initscripts/legacy-actions/nginx/check-reload
/usr/libexec/initscripts/legacy-actions/nginx/upgrade
/usr/share/nginx
/usr/share/nginx/html
/usr/share/nginx/html/50x.html
/usr/share/nginx/html/index.html
```
#### Nginx查看版本和查看编译参数
```bash
# 查看Nginx版本
nginx -v
# 查看编译参数和Nginx版本
Nginx -V
# -V 输出内容如下
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

#### nginx配置语法
在/etc/nginx下存在两个文件，分别是/etc/nginx/conf.d/default.conf和/etc/nginx/nginx.conf.
我们打开nginx.conf文件，内容如下
```bash
# 设置nginx服务的系统使用用户
user  nginx;  
# 工作进程数
worker_processes  1;

# nginx的错误日志
error_log  /var/log/nginx/error.log warn;
# nginx服务启动时候的pid
pid        /var/run/nginx.pid;

# 事件
events {
# 每个进程允许最大连接数
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65; 
    include /etc/nginx/conf.d/*.conf;
}
```
我们会发现，他有一条内容是 include /etc/nginx/conf.d/*.conf,其实nginx会扫描conf.d文件夹下面的所有.conf的文件

```bash
# nginx默认配置语法 地址： /etc/nginx/conf.d/default.conf 
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## 聊一聊Nginx下中各个模块的配置
接下来我们要用大篇幅的内容来介绍一下nginx中各个模块的配置，在这之前我们需要了解一下Nginx中整个配置架构。
根据上面的nginx.conf和default.conf配置文件，我们可以知道nginx大致的配置架构为  大体是一个conf配置文件，在文件的开头会有一些全局配置，如设置用户 设置报错日志存储的地址 设置服务器启动时候的pid 在这之后则是多个http，在每个htt品种又包含了多个server的配置。在nginx.conf文件中我们只见到了http这一层，并且他指向了一个/etc/nginx/conf.d/*.conf的文件合集，其中的文件default.conf中则包含了server层
#### 全局配置层

#### http层

#### server层



