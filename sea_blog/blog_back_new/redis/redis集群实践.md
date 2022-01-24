---
title: redis集群实践
categories:
  - redis
tags:
  - redis
abbrlink: 5e2995eb
date: 2020-10-30 18:04:01
---


<!-- more -->


# redis集群实践

## redis分区规则
分布式数据库首先要解决把整个数据集按照分区规则映射到多个节点的问题，即把数据集划分到多个节点上，每个节点负责整体数据的一个子集。
![2020-11-12-16-29-30](http://noback.upyun.com/2020-11-12-16-29-30.png)

### 分区规则
分区规则主要有`顺序分区`和`哈希分区`,在redis-cluster集群当中，主要使用的是哈希分区
哈希分区主要包括`节点取余分区`和`一致性哈希分区`

一致性哈希分区规则如下，为系统中每个节点分配一个token，范围一般在0~232，这些token构成一个哈希环。数据读写执行节点查找操作时，先根据key计算hash值，然后顺时针找到第一个大于等于该哈希值的token节点
![2020-11-12-16-38-36](http://noback.upyun.com/2020-11-12-16-38-36.png)
把key值通过hash之后存放到每一个插槽里面
![2020-11-12-16-39-49](http://noback.upyun.com/2020-11-12-16-39-49.png)


### 一致性哈希的优缺点
优点
加入和删除节点只影响哈希环中相邻的节点，对其他节点无影响
缺点
- 加减节点会造成哈希环中部分数据无法命中，需要手动处理或者忽略这部分数据，因此一致性哈希常用于缓存场景
- 当使用少量节点时，节点变化将大范围影响哈希环中数据映射，因此这种方式不适合少量数据节点的分布式方案
- 普通的一致性哈希分区在增减节点时需要增加一倍或减去一半节点才能保证数据和负载的均衡

后面有了虚拟槽区位

### 虚拟槽分区
虚拟槽分区巧妙地使用了哈希空间，使用分散度良好的哈希函数把所有数据映射到一个固定范围的整数集合中，整数定义为槽（slot）
槽位的数量一定大于节点的数量，每一个节点掌管一定数量的槽位
比如 假设一个集群有5个节点，每个节点平均大约负责3276个槽。由于采用高质量的哈希算法，每个槽所映射的数据通常比较均匀，将数据平均划分到5个节点进行数据分区。Redis Cluster就是采用虚拟槽分区

key值哈希计算方法
```bash
slot=CRC16（key）&16383
```
所有的key根据哈希函数映射到0~16383整数槽内
多个节点平均分配16384个槽
![2020-11-12-16-46-13](http://noback.upyun.com/2020-11-12-16-46-13.png)
key根据哈希算法分配到16384个槽中
![2020-11-12-16-46-41](http://noback.upyun.com/2020-11-12-16-46-41.png)

特点
解耦数据和节点之间的关系，简化了节点扩容和收缩难度。
节点自身维护槽的映射关系，不需要客户端或者代理服务维护槽分区元数据。
支持节点、槽、键之间的映射查询，用于数据路由、在线伸缩等场景。

## redis-cluster常用命令
```bash
# 登录
redis-cli -c -p 6382 -h 10.0.6.55
# 打印集群信息
cluster info   # 只有当16384个槽位全部被分配之后，才会显示OK
# 显示当前节点在集群中的id
cluster myid
# 列出集群当前已知的所有节点， 以及这些节点的相关信息
cluster nodes
# 握手 将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子
cluster meet <ip> <port>
# 从集群中移除 node_id 指定的结点  (线上不建议使用 建议使用redis-cli --cluster del-node)
cluster forget <node_id>
# 将当前从节点设置为 node_id 指定的master节点的slave节点。只能针对slave节点操作
cluster replicate <master_node_id>
# 将节点的配置文件保存到硬盘里面
cluster saveconfig 

# 槽(slot)
# 将一个或多个槽(slot)指派给当前节点
cluster addslots <slot> [slot ...]
# 移除一个或多个槽对当前节点的指派
cluster delslots <slot> [slot ...] 
# 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点
cluster flushslots 
# 迁移节点
# 从目标节点导入
cluster setslot <slot> importing <target_node_id>
# 将本节点的槽 slot 迁移到 node_id 指定的节点中
cluster setslot <slot> migrating <target_node_id>
# 取消对槽slot的导入或者迁移
cluster setslot <slot> stable 
# 键hash 
# 计算键key应该被放置在哪个槽上
cluster keyslot <key>
# 返回槽 slot 目前包含的键值对数量
cluster countkeysinslot <slot>
# 返回 count 个 slot 槽中的键 
cluster getkeysinslot <slot> <count>
```

## redis集群的存储方式
redis会把存储的key值做一个hash 然后存放到各自的桶里面
reids一共分成16384个桶，也就是说最多可以有16384个master 也就是16384*2个机器(一主一从)

如下所示，将不同key存放到不同的桶当中  
集群如下
![2020-11-04-12-02-16](http://noback.upyun.com/2020-11-04-12-02-16.png)

```bash
# 获取当前key被hash后的id是多少
cluster keyslot name # 5798 所以name这个key应该被放在5798这个槽里面  根据上面这个图的分配，可以看出name应该存放在6380这个端口下面
```
![2020-11-04-16-37-05](http://noback.upyun.com/2020-11-04-16-37-05.png)

如果当前设置的key不在当前的端口下，则会报错
![2020-11-04-16-38-03](http://noback.upyun.com/2020-11-04-16-38-03.png)
比如要在6379下面才能去设置
当然这是单纯的命令行，在程序中会有其他的解决办法




1. 槽这个东西的是怎样的，因为key是被hash后的，所以这个槽里面可以一直放数据嘛
2. hash的算法是固定的嘛 ，假设有两个集群，那么同样的key  name 他所得到的桶id是一样的嘛


### redis-cluster 获取桶内数据
redis-cluster 支持在指定的桶(根据ID)中获取指定数量的单位
比如现在我们往指定的桶中存放指定一定数量的key-value
![2020-11-04-16-48-20](http://noback.upyun.com/2020-11-04-16-48-20.png)
获取桶ID为858中的"java"

### HashTag使用

Redis 集群没有使用一致性hash, 而是引入了 哈希槽的概念，预分好16384个桶，根据 CRC16(key) mod 16384的值，决定将一个key放到哪个桶中，每个Redis物理结点负责一部分桶的管理，当发生Redis节点的增减时，调整桶的分布即可。

为了实现将key分到相同机器，就需要相同的hash值，即相同的key，但key相同是不现实的，因为key都有不同的用途。例如user:user1:ids保存用户id，user:user1:detail保存用户的具体信息，两个key不可能同名。两个key其实有相同的地方，即user。能不能拿这一部分去计算hash呢？这就是 Hash Tag。允许用key的部分字符串来计算hash,
当一个key包含 {} 的时候，就不对整个key做hash，而仅对 {} 包括的字符串做hash。

假设hash算法为sha1。对user:{user1}:ids和user:{user1}:detail，其hash值都等同于sha1(user1)。
将不同的key放在同一个槽之内，如下
![2020-11-04-17-02-47](http://noback.upyun.com/2020-11-04-17-02-47.png)
在hash计算的时候，回去计算a的hash值，然后放到对应id的桶当中。


## 手动操作槽与数据的迁移
![2020-11-04-12-02-16](http://noback.upyun.com/2020-11-04-12-02-16.png)
首先准备数据，准备3条数据到6379中，操作如下
```bash
mset {test1}name hzj {test1}name2 cy {test1}name3 helloworld
cluster keyslot test1 # 获取桶ID
cluster getkeysinslot 4768 5 # 获取对应桶ID中对应数量的值
```
![2020-11-04-17-16-03](http://noback.upyun.com/2020-11-04-17-16-03.png)
现在我们将6379节点4768桶中的数据转移到6380当中去
```bash
# 目标节点准备导入桶4768中的数据
10.0.6.35:6380> cluster setslot 4768 IMPORTING 2a46fd184a14f6d0e320fdd31cfd93c6d80bf052
# 源节点 准备导出桶4786中的数据
10.0.6.35:6379> cluster setslot 4768 MIGRATING f3f2c1910f30573d82071ee1746b3f3b0a07267d
# 确认导出状态
10.0.6.35:6380> cluster nodes | grep master
# 批量获取对应槽内的key-value
10.0.6.35:6379> cluster getkeysinslot 4768 100
# 批量迁移
10.0.6.35:6379> migrate 127.0.0.1 6380 "" 0 5000 KEYS {test1}name {test1}name2 {test1}name3
# 通知其他节点槽4768 指派给节点
iredis -h 10.0.6.35 -p 6379 cluster setslot 4768 node f3f2c1910f30573d82071ee1746b3f3b0a07267d
iredis -h 10.0.6.35 -p 6381 cluster setslot 4768 node f3f2c1910f30573d82071ee1746b3f3b0a07267d
iredis -h 10.0.6.35 -p 6382 cluster setslot 4768 node f3f2c1910f30573d82071ee1746b3f3b0a07267d
# 查看节点 是否转移成功

```
确认导出状态的图
![2020-11-04-17-22-21](http://noback.upyun.com/2020-11-04-17-22-21.png)
转移成功状态图
![2020-11-04-17-56-11](http://noback.upyun.com/2020-11-04-17-56-11.png)


## 脚本操作槽与数据的迁移
创建环境
![2020-11-04-18-04-30](http://noback.upyun.com/2020-11-04-18-04-30.png)
![2020-11-04-18-05-08](http://noback.upyun.com/2020-11-04-18-05-08.png)

### 老版本使用ruby脚本
```bash
redis-trib.rb reshard host:port --from <arg> --to <arg> --slots <arg> --yes --timeout <arg> --pipeline <arg>
```
- host 集群内任意节点地址，用来获取参数
- --form 制定源节点的id，如果有多个源节点，使用逗号分隔，如果是all源节点变为集群内所有主节点，在迁移过程中提示用户输入
- --to  需要迁移的目标节点的id，目标节点只能填写一个，在迁移过程中提示用户输入。
- --slots 需要迁移槽的总数量，在迁移过程中提示用户输入。
- --yes  当打印出reshard执行计划时，是否需要用户输入yes确认后再执行reshard。
- --timeout 控制每次migrate操作的超时时间，默认为60000毫秒。
- --pipline 控制每次批量迁移键的数量，默认为10

### 新版本使用redis-cli脚本
```bash
./redis-cli --cluster reshard 127.0.0.1 6379
```
使用参数和ruby脚本一样


### 添加从节点
```bash
10.0.6.35:6386> cluster replicate 16e43ced3ae6c4fd7912b7f609df363ab56cd89a
```

## 节点下线流程
要保证节点的下线，首先是要转移节点中的槽，保证数据的完整性
要通知所有的其他节点， 该节点已经下线了

### 忘记节点
```bash
cluster forget{down NodeId}
```
当节点接收到cluster forget{down NodeId}命令后，会把nodeId指定的节点加入到禁用列表中，在禁用列表内的节点不再发送Gossip消息。禁用列表有效期是60秒，超过60秒节点会再次参与消息交换。也就是说当第一次forget命令发出后，我们有60秒的时间让集群内的所有节点忘记下线节点。

下线需求，最好先下线从节点，再下线主节点。如果从节点丢失了主节点，他会自动去更新一个新的主节点，这样会出现没有必要的全量复制
线上环境建议使用
```bash
# 旧版本
redis-trib.rb del-node{host：port}{downNodeId}
# 新版本
redis-cli --cluster del-node {host:port}{downNodeId}
# 示例
./redis-cli --cluster del-node  127.0.0.1:6379 81b2d20ffe75f429b4934b494e368697b388ff22
```



### 节点下线实验
当前情况如下
![2020-11-05-10-10-29](http://noback.upyun.com/2020-11-05-10-10-29.png)
现在要下线6381节点，将6381中的槽平均分配到其他节点中去

将1365个槽转移到6385后
![2020-11-05-10-21-35](http://noback.upyun.com/2020-11-05-10-21-35.png)
![2020-11-05-10-17-19](http://noback.upyun.com/2020-11-05-10-17-19.png)
全部转移之后
![2020-11-05-10-25-18](http://noback.upyun.com/2020-11-05-10-25-18.png)
下线6381主节点和6384从节点
![2020-11-05-10-33-14](http://noback.upyun.com/2020-11-05-10-33-14.png)

## 请求重定向
在集群模式下，Redis接收任何键相关命令时首先计算键对应的槽，再根据槽找出所对应的节点，如果节点是自身，则处理键命令；否则回复MOVED重定向错误，通知客户端请求正确的节点。这个过程称为MOVED重定向
内容如下

![2020-11-05-11-02-01](http://noback.upyun.com/2020-11-05-11-02-01.png)
![2020-11-05-11-02-16](http://noback.upyun.com/2020-11-05-11-02-16.png)

name这个key的hash值计算出来是5798 应该是在6380这个节点上面的，但是当前的节点是6386这个节点，于是在设置值的时候回出现重定向。这里只做出反应，并不作出转发的动作，因此不会去6380上面设置key


## 快速部署脚本

ruby安装以及gem源更新成国内源

```bash
wget https://cache.ruby-china.com/pub/ruby/ruby-2.6.5.tar.xz
xz -d ruby-2.6.5.tar.xz         
tar -xvf ruby-2.6.5.tar   
cd /home/joyce/soft/ruby-2.6.5
./configure  && make -j8 && make install && ruby -v
# 执行配置。或者：   ./configure  --with-openssl-dir=/usr/local/ssl  可以解决报错：Unable to require openssl, install OpenSSL and rebuild ruby (preferred) or use non-HTTPS sources

gem sources 　　# 查看当前源
gem -s sources  http://mirrors.aliyun.com/rubygems/  # 添加阿里源
gem sources -r https://rubygems.org/  # 删除默认源地址
gem sources -u 　　 # 更新源的缓存
```

redis中自带ruby脚本，可以用来快速搭建redis集群，脚本位置在 `/redis/src/redis-trib.rb`
<font color='red'>5.0的redis中redis-trib这个脚本已经被弃用了，工能都放在redis-cli里面</font>

机器部署
```bash
#!/bin/bash

module_file="./redis.conf"

if test -z "$1"
then
  echo "no set config name"
  exit 1
else
  echo "config the redis-$1.conf now"
  Port="$1"
fi


if test -f "./redis-$1.conf"
then
  echo "file exit"
else
  echo "copy $module_file >> ./redis-$1.conf"
  `cp $module_file ./redis-$1.conf`
fi

sed -i -e  "/^port/s/.*/port ${Port}/"  \
    -e "/^daemonize/s/no/yes/"   \
    -e "/^logfile/s/.*/logfile \"${Port}.log\"/" \
    -e "/^dbfilename/s/.*/dbfilename \"dump-${Port}.rdb\"/"   \
    -e "/^dir/s@.*@dir ./data/@" \
    -e 's/^bind/#&/' \
    -e "/^protected-mode/s/yes/no/" \
    -e "/cluster-node-timeout/s@# @""@" \
    -e "/cluster-config-file/s@.*@cluster-config-file nodes-${Port}.conf@"\
    -e "/cluster-enabled/s@# @""@" \
    redis-${Port}.conf

echo "success"
# sh ./master.sh 6666
```


## 错误预告

### 没有设置集群模式
```bash
Redis报错：ERR This instance has cluster support disabled
```
配置没有修改成集群模式
```bash
# cluster-enabled yes  =>
cluster-enabled yes
```

### 迁移未完成
![2020-11-04-18-26-28](http://noback.upyun.com/2020-11-04-18-26-28.png)
![2020-11-04-18-26-56](http://noback.upyun.com/2020-11-04-18-26-56.png)
一般情况下是手动迁移没有完成
解决办法，关闭迁移
```bash
cluster setslot <slot_id> STABLE
```


## 单点集群实验测试案例
开启节点集群模式
标注的几个是需要修改的

```bash
protected-mode no
port 6666  # "this"
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile "6666.log"  # "this"
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump-6666.rdb"  # "this"
dir ./data/
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
cluster-enabled yes   # "this"
cluster-config-file nodes-6666.conf   # "this"
cluster-node-timeout 15000   # "this"
slowlog-log-slower-than 10000 
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
```

启动集群
```bash
redis-server conf/redis-6379.conf
redis-server conf/redis-6380.conf
redis-server conf/redis-6381.conf
redis-server conf/redis-6382.conf
redis-server conf/redis-6383.conf
redis-server conf/redis-6384.conf
```
![2020-11-12-17-27-57](http://noback.upyun.com/2020-11-12-17-27-57.png)

节点握手，当前只是每一个redis启动了集群模式，接下来还要进行握手连接