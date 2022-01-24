---
title: redis优化
date: 2020-11-09 17:19:39
categories: [redis,]  
tags: [redis,]
---


<!-- more -->

# redis优化

```bash
cat redis-base.conf 
daemonize no # 使用supervisor管理redis 可以自动重启
bind 0.0.0.0  # 支持远程管理
timeout 300  
loglevel warning
databases 16
save 86400 1  #  触发bgsave 存储到rdb 这里是每次出现数据变化，就保存一次
rdbcompression yes # 使用LZF算法对rdb文件进行压缩，如果要节省一些CPU，可以设置为no。
appendfsync no
no-appendfsync-on-rewrite yes
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
activerehashing yes
maxmemory 3000000000
maxmemory-policy volatile-lru
slave-serve-stale-data yes
appendonly yes

cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000

cat redis-6379.conf 
include /usr/local/redis/etc/yupoo/redis-base.conf

pidfile /var/run/redis-6379.pid
port 6379
logfile /disk/ssd1/logs/redis/redis-6379.log
dbfilename redis-6379.rdb
dir /disk/ssd1/redisdb/yupoo/6379
```




```bash
daemonize no # 使用supervisor管理redis 可以自动重启 注意配置成守护进程后Redis会将进程号写入文件/var/run/redis.pid
bind 0.0.0.0  # 支持远程管理
timeout 300  # 客户端空闲多少秒后关闭连接(0为不关闭)
loglevel warning # 配置日志级别
databases 16 # 数据库的数量 默认为16个
save 86400 1  #  触发bgsave 存储到rdb
rdbcompression yes # 使用LZF算法对rdb文件进行压缩，如果要节省一些CPU，可以设置为no。
# Redis支持三种不同的模式：
# no：不要立刻刷，只有在操作系统需要刷的时候再刷。比较快。
# always：每次写操作都立刻写入到aof文件。慢，但是最安全。
# everysec：每秒写一次。折中方案。
appendfsync no # AOF模式
no-appendfsync-on-rewrite yes # 关闭延迟
hash-max-ziplist-entries 512 # 键值对哈希存储方式
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
activerehashing yes 
maxmemory 3000000000 # 最大内存
maxmemory-policy volatile-lru 
slave-serve-stale-data yes  # 当slave 失去连接的时候,继续响应客户端的请求
appendonly yes #开启AOF

```