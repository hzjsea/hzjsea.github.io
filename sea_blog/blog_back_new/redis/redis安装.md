---
title: redis安装
categories:
  - redis
tags:
  - redis
abbrlink: 7b25d017
date: 2020-05-18 11:13:34
---

![2020-10-28-17-47-35](http://noback.upyun.com/2020-10-28-17-47-35.png)
<!-- more -->

# redis使用
Redis是开源的(BSD许可)内存数据结构存储,用作数据库,缓存和消息代理.它支持数据结构
Redis不是简单的键值存储，它实际上是一个数据结构服务器，支持不同类型的值。这意味着在传统键值存储中，您将字符串键与字符串值相关联，而在Redis中，该值不仅限于简单的字符串，还可以保存更复杂的数据结构


## redis安装
### redis本地安装
```bash
mkdir /root/package
yum install wget  epel-release -y
wget http://download.redis.io/releases/redis-5.0.8.tar.gz -O /root/package/redis-5.0.8.tar.gz 
tar zxvf /root/package/redis-5.0.8.tar.gz -C /usr/local/
cd /usr/lcoal/redis-5.0.8/src && make distclean && make && make install

# 运行redis 
./usr/local/redis-5.0.8/src/redis-server
# 做软连接到/usr/local/bin下面
ln -s /usr/local/redis-5.0.8/src/redis-server /usr/local/bin/redis-server
ln -s /usr/local/redis-5.0.8/src/redis-cli /usr/local/bin/redis-cli
ln -s /usr/local/redis-5.0.8/src/redis-sentinel /usr/lcoal/bin/redis-sentinel
```

编译到其他目录
```bash
cd /root/package/redis-5.0.8.tar.gz
make PREFIX=/package/root/ install
```

![2020-05-21-10-32-37](http://noback.upyun.com/2020-05-21-10-32-37.png)

### redis后台运行
redis后台运行有两种方法，一种是直接跑，另外一种是挂起式
直接跑的形式
```bash
nohup ./src/redis-server & 
```
另外一种是通过修改redis.conf，运行的时候需要指定redis.conf
```bash
sed -i "/^daemonize/s/no/yes/" /usr/local/redis-5.0.8/redis.conf
./src/redis-server redis.conf
# 查看后台进程
lsof -i :6379
# 杀死后台进程
lsof -i :6379 | cut -d " " -f 2 | xargs -I {} kill -9 {}
```
### redis 远程登录
需要修改redis.conf文件
```bash
sed -i 's/^bind/#&/' /usr/local/redis-5.0.8/redis.conf
sed -i "/^protected-mode/s/yes/no/" /usr/lcoal/redis-5.0.8/redis.conf
```
protected-mode：您确定希望其他主机的客户端连接到Redis，即使未配置身份验证，也不配置特定的接口集

### 设置密码
```bash
sed -n "/^#/s/# requirepass/requirepass 123456/p"  /usr/local/redis-5.0.8/redis.conf
```
如果是以服务方式启动，设置密码后还需要修改脚本/etc/init.d/reds_6379，否则会报错无权限
```bash
REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli
# 新增下面一行
PASSWORD=123456
```
修改shutdown命令
```bash
# CLIEXEC -p $REDISPORT shutdown
CLIEXEC -a $PASSWORD -p $REDISPORT shutdown
```

### 设置为服务
有两种方法，一种是自己手动去设置，另外一种是使用redis自带去设置
```bash
mkdir /etc/redis
cp /usr/local/redis-5.0.8/redis.conf /etc/redis/6379.conf
cp /usr/local/redis-5.0.8/utils/redis_init_script /etc/init.d/redis_6379
```
启动和关闭redis
```bash
systemctl start redis_6379.service
systemctl stop redis_6379.service
```
设置开机启动
```bash
chkconfig redis_6379 on
chkconfig --list
```

自带的redis去设置
```bash
shell> utils/install_server.sh 
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf] 
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log] 
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379] 
Selected default - /var/lib/redis/6379
Please select the redis executable path [/usr/local/bin/redis-server] 
Selected config:
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /var/log/redis_6379.log
Data dir       : /var/lib/redis/6379
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
/var/run/redis_6379.pid exists, process is already running or crashed
Installation successful!
```



### redis docker安装
```bash
docker pull redis
docker run -p 6379:6379 --name myredis -d redis redis-server --appendonly yes

# --name 是启动后容器对应的名称
# -p 是暴露docker内部的端口映射到本地服务器的端口
# redis redis-server：redis 代表着 redis 镜像 redis-server 表示的是执行的命令，也是就 redis 的启动命令，跟我们 linux 下面的 ./redis-server 一样
# --appendonly yes：开启 AOF 持久化


# 使用docker-redis
docker exec -it dockerRedis redis-cli
```

## redis工具
自从用了mycli以后，就喜欢上这个交互式命令工具了，这个工具有智能提示，对于一开始学习记不住命令有很大帮助
github地址
https://github.com/laixintao/iredis/
安装流程
```bash
# 开启虚拟环境
pipenv shell
# 安装工具
pip install iredis
# 连接远程redis
iredis -h 10.0.43.106 -p 6379
```
### redis可视化工具
- medis
https://github.com/luin/medis
- tableplus 
https://tableplus.com/


## 报错内容附录
### gcc没有安装
```bash
make cc Command not found
```
这是因为系统中没有安装gcc的问题，解决方法是安装gcc目录
```bash
yum install gcc -y
```

### 编译缓存没有清理
```bash
jemalloc/jemalloc.h: No such file or directory。
```
这是因为之前编译的时候，编译出错了，缓存包就停留在目标文件当中，这时候二次编译的时候就会出现错误了
解决方法 清理缓存包
```bash
make distclean  && make && make install 
```
