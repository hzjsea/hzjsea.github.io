---
title: redis哨兵实践
categories:
  - redis
tags:
  - redis
abbrlink: 878cfa9b
date: 2020-10-29 16:11:28
---

![2020-10-28-17-47-35](http://noback.upyun.com/2020-10-28-17-47-35.png)
<!-- more -->
# redis哨兵实践

## redis主从复制
普通情况下
Redis主从复制模式下，一旦节点出现故障不能提供服务，需要人工去晋升从节点变成主节点，同时还要通知应用方更新主节点地址，对于很多应用场景这种故障处理的方式是无法接受的。

![2020-10-30-11-43-31](http://noback.upyun.com/2020-10-30-11-43-31.png)

### 无哨兵的主从复制优缺点
优点
Redis的主从复制模式可以将主节点的数据改变同步给从节点
- 作为主节点的一个备份，一旦主节点出了故障不可达的情况，从节点可以作为后备“顶”上来，并且保证数据尽量不丢失(主从复制是最终一致性)
- 从节点可以扩展主节点的读能力,一旦主节点不能支撑住大并发量的读操作，从节点可以在一定程度上帮助主节点分担读压力

缺点
- 一旦主节点出现故障，就要手动干预主从关系。
- 主节点的写能力受到单机的限制
- 主节点的存储能力受到单机的限制

![2020-10-30-11-43-43](http://noback.upyun.com/2020-10-30-11-43-43.png)
![2020-10-30-11-45-43](http://noback.upyun.com/2020-10-30-11-45-43.png)
一旦master所在的机器出现故障，master下面的从机会进行选举，最后选举出来一个new-master,等到old-master恢复之后，old-master又会重新获取master的位置，new-master从新变成从机


### 使用哨兵的主从复制
哨兵的出现，增加了redis的高可用性，当主节点出现故障的时候，redis Sentinel能自动完成故障发现和故障转移，并通知应用方，从而实现真正的高可用

哨兵节点的作用流程，这个哨兵的的架构如下图所示
![2020-10-30-11-50-02](http://noback.upyun.com/2020-10-30-11-50-02.png)
Redis Sentinel是一个分布式架构，其中包含若干个Sentinel节点和Redis数据节点，
每个Sentinel节点会对数据节点和其余Sentinel节点进行监控，当它发现节点不可达时，会对节点做下线标识。
如果被标识的是主节点，它还会和其他Sentinel节点进行“协商”，当大多数Sentinel节点都认为主节点不可达时，它们会选举出一个Sentinel节点来完成自动故障转移的工作，同时会将这个变化实时通知给Redis应用方。整个过程完全是自动的，不需要人工来介入，所以这套方案很有效地解决了Redis的高可用问题。

![2020-10-30-11-52-29](http://noback.upyun.com/2020-10-30-11-52-29.png)

1. 哨兵节点各司其职，他们监控的内容主要有三点，一个是哨兵节点群内部，互相监控，一个是master节点，一个是slave节点
2. 当master出现故障的时候，哨兵节点发现master主节点不可达
3. 当达到一定数量(配置完成)的Sentinel节点发现主节点不可达的时候，哨兵节点群内部就会协商出来一个`领导Sentinel`,这个哨兵节点主要负责的工作是进行故障转移
4. 故障转移的主要内容是 `slaveof no one `对一个从属服务器执行命令，将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得数据集不会被丢弃，这样做可以使得整个切换的流程不间断运行。选择一个slave机器变成master主机。同时领导哨兵还会通知给client
5. 重新回归到1的模式

哨兵主从复制的几个名词
- 监控
Sentinel节点会定期检测Redis数据节点，其余Sentinel节点是否可达
- 通知
Sentinel节点会讲故障转移到结果通知给应用方
- 主节点故障转移
实现从节点晋升为主节点并维护后续正确的主从关系
- 配置提供者
在Redis Sentinel结构中，客户端初始化的时候，连接的是Sentinel节点集合，从中获取主节点信息。节点的故障判断是由多个Sentinel节点共同完成，这样可以有效地防止误判，Sentinel节点集合是由多个Sentinel组合而成，这样即使个别Sentinel节点不可用，整个Sentinel节点集合依然是健壮存在的

<font color='red'>Sentinel节点是独立的Redis节点，他们不存储数据，只支持部分命令</font>

### Redis-Sentinel实践
这里用的单机做实验，但事实上应该是放在一个集群里面的，如果所有的架构都放在一台服务器下面的话，高可用性就不存在了。单机挂了，这个集群就挂了，同时存储量也上不去。

主节点配置
```bash
redis-6379.conf
port 6379  
daemonize yes  
logfile "6379.log"
dbfilename "dump-6379.rdb"  
dir "/opt/soft/redis/data/"
```

从节点配置
```bash
redis-6380.conf
port 6380
daemonize yes  
logfile "6380.log"
dbfilename "dump-6380.rdb"  
dir "/opt/soft/redis/data/"
slaveof 127.0.0.1 6380
```

哨兵节点配置
```bash
redis-sentinel-26379.conf
port 26379 
# 是否后台运行
daemonize yes
# 日志存储地点
logfile "26379.log"  
# 节点工作地点
dir /opt/soft/redis/data 
# 监控名字叫master的主机，地址为127.0.0.1 端口为6379 至少2个哨兵节点同时检测不到才会被判断为故障
# 一般设置为Sentinel节点的一半+1
sentinel monitor master 127.0.0.1 6379 2  

sentinel down-after-milliseconds master 30000  
sentinel parallel-syncs master 1  
sentinel failover-timeout master 180000  

# 如果Sentinel监控的主节点配置了密码，sentinel auth-pass配置通过添加主节点的密码，防止Sentinel节点对主节点无法监控
# sentinel auth-pass <master-name> <password>

# sentinel notification-script的作用是在故障转移期间，当一些警告级别的Sentinel事件发生（指重要事件，例如-sdown：客观下线、-odown：主观下线）时，会触发对应路径的脚本，并向脚本发送相应的事件参数
# sentinel notification-script
```


启动节点
```bash
redis-server redis-6379.conf
# 判断节点是否启动成功
redis-cli -h 127.0.0.1 -p 6379 ping 
# Pong
```

启动哨兵节点
```bash
# 方法1
redis-server redis-sentinel-26379.conf --sentinel
# 方法2
redis-sentinel redis-sentinel-26379.conf

```

确认主从关系
```bash
[root@dev workspace]# ./redis-cli -h 127.0.0.1 -p 6379 info replication
# Replication
role:master    # 主机名字
connected_slaves:2  # 连接了2个从机
slave0:ip=127.0.0.1,port=6380,state=online,offset=2238145,lag=1  # 第一个从机的信息
slave1:ip=127.0.0.1,port=6381,state=online,offset=2238145,lag=1  # 第二个从机的信息
master_replid:7b6386f55b675e16c73fd26e0462422b56f88642   
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:2238276
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1189701
repl_backlog_histlen:1048576
[root@dev workspace]# ./redis-cli -h 127.0.0.1 -p 6380 info replication
# Replication 
role:slave   # 从机名字
master_host:127.0.0.1  # 主机地址
master_port:6379  # 主机端口
master_link_status:up # 主机状态
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:2257673
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:7b6386f55b675e16c73fd26e0462422b56f88642
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:2257673
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1209098
repl_backlog_histlen:1048576
[root@dev workspace]# ./redis-cli -h 127.0.0.1 -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=master,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```


### 哨兵节点配置优化
在整个架构开启之前，redis-sentinel的配置内容如下
```bash
redis-sentinel-26379.conf
port 26379 
daemonize yes
logfile "26379.log"  
dir /opt/soft/redis/data 
sentinel monitor master 127.0.0.1 6379 2  
# 判断节点不可达的时间，虽然是以<master-name>为参数，但实际上对Sentinel节点，主节点，从节点同时有效
sentinel down-after-milliseconds master 30000 
# 当主节点出现判定的时候，Sentinel会执行故障转移操作
# 转移操作中复制操作可以做多个同时进行复制，如果是3 则是3个同步复制，会出现阻塞或加大磁盘IO开销
sentinel parallel-syncs master 1  
# 故障转移超时时间
sentinel failover-timeout master 180000   
```
上面只写了master的配置，但事实上，sentinel节点还会监控其他的slave节点，这是动态更新在配置文件中的，如下
```bash
sentinel myid 7385f91ceab39c12e000af580d9a67624abb8387
sentinel deny-scripts-reconfig yes
sentinel monitor master 127.0.0.1 6379 2
sentinel config-epoch master 0
# Generated by CONFIG REWRITE
sentinel leader-epoch master 0   # master
sentinel known-replica master 127.0.0.1 6381  # slave 1
sentinel known-replica master 127.0.0.1 6380  # slave 2 
sentinel known-sentinel master 127.0.0.1 26380 d4d9781b5df7f030d36743b66c7c61b3c7a5cdd5 # sentinel2
sentinel known-sentinel master 127.0.0.1 26381 050e3625a856a0fe34750b62ded4751cc975ee2f # sentinel3
sentinel current-epoch 0
```

### 选举条件
Sentinel的选举条件
```bash
sentinel monitor master 127.0.0.1 6379 2(quorum)
```
一般设置为`(主节点数量/2)+1`
选举条件 至少要有 (quorum,num(sentinel)/2+1) 个Sentinel节点参与选举，才能选出领导者Sentinel

### 故障转移操作那个
1) 选出合适的从节点
2) 晋升选出的从节点为主节点
3) 命令其余从节点复制新的主节点
4) 等到原主节点恢复后命令他去复制新的主节点

- 如果Redis Sentinel对一个主节点故障转移失败，那么下次再对该主节点做故障转移的起始时间是failover-timeout的2倍
- 在2）阶段时，如果Sentinel节点向1）阶段选出来的从节点执行slaveof no one一直失败（例如该从节点此时出现故障），当此过程超过failover-timeout时，则故障转移失败
- 在2）阶段如果执行成功，Sentinel节点还会执行info命令来确认1）阶段选出来的节点确实晋升为主节点，如果此过程执行时间超过failover-timeout时，则故障转移失
- 如果3）阶段执行时间超过了failover-timeout（不包含复制时间），则故障转移失败。注意即使超过了这个时间，Sentinel节点也会最终配置从节点去同步最新的主节点。


### 监控多个主节点
master节点与slave节点配置不变
sentinel节点配置如下
```bash
sentinel monitor master-business-1 10.10.xx.1 6379 2 
sentinel down-after-milliseconds master-business-1 60000 
sentinel failover-timeout master-business-1 180000 
sentinel parallel-syncs master-business-1 1
sentinel monitor master-business-2 10.16.xx.2 6380 2 
sentinel down-after-milliseconds master-business-2 10000 
sentinel failover-timeout master-business-2 180000 
sentinel parallel-syncs master-business-2 1
```

![2020-10-30-15-09-12](http://noback.upyun.com/2020-10-30-15-09-12.png)


### 动态设置配置参数
```bash
sentinel set master quorum 2
# 设置故障判断时间
sentinel set master down-after-milliseconds 30000
# 设置故障转移超时时间
sentinel set master failover-timeout 360000
# 设置故障转移同时复制数量
sentinel set master parallel-syncs 2
# 设置通知脚本
sentinel set master notification-script /opt/xx.sh
# 设置密码
sentinel set master auth-parallel password
```

### 部署技巧 以及API
- 哨兵节点不应该部署在同一个物理机上面， 应该是一个Redis+一个哨兵节点 一个用于存储 一个用于监控
- 至少三个且奇数个数的Sentinel节点，提高准确性

```bash
# 登录Sentinel
./redis-cli  -h 127.0.0.1 -p 26379
# 查看masters数量
sentinel masters
# 查看主节点信息
sentinel master <master-name>
# 查看特定主节点下面的从节点信息
sentinel slaves <master-name>
# 查看哨兵节点信息
sentinel sentinels <master-name>
# 返回主节点的IP地址和端口
sentinel get-master-addr-by-name <master-name>
# 重置哨兵节点，清除相关状态，比如故障转移记录
sentinel reset <master-name>
# 对主节点进行强制故障
sentinel failover <master-name>
# 检查当前可达哨兵节点数量为几个 比如quorum=3 命令的结果是2  则无法故障转移
sentinel ckquorum <master-name>
# 将Sentinel节点的配置强制刷到磁盘上
sentinel flushconfig
# 取消当前Sentinel对主节点的监控
sentinel remove<master-name>
# 设置Sentinel对主节点的监控
sentinel monitor master
```


### 客户端连接
slave会根据sentinel的协商之后，生成master节点，因此master的IP和端口会不断的变化
于是客户端连接就出现了问题，如果主节点挂了，Redis Sentinel虽然可以完成故障转移，但是客户端无法获取这个变化。
因此如果想要正确的连接Redis Sentinel,必须有Sentinel节点集合和masterName两个参数。

### 客户端连接实现原理
1. 遍历Sentinel节点集合获取一个可用的Sentinel节点
2. 通过`sentinel get-master-addr-by-name <master-name>` 获取主节点信息
3. 验证当前获取的"主节点"是否为真正的主节点，这样做的目的是为了防止故障转移期间主节点的变化。
4. 保持和Sentinel节点集合的“联系”，时刻获取关于主节点的相关“信息”

java连接 redis实例

python连接 redis实例
```py
import redis
from redis.sentinel import  Sentinel


sen = Sentinel([('10.0.6.55',26379),('10.0.6.55',26380),('10.0.6.55',26381)],socket_timeout=0.5)

master = sen.discover_master('master')
print(master)

slaves = sen.discover_slaves('master')
print(slaves)

# 查看master
master = sen.discover_master('master')  # ('127.0.0.1', 6379)

# 查看哨兵节点
slaves = sen.discover_slaves('master')  # [('127.0.0.1', 6380), ('127.0.0.1', 6381)]

# 连接信息
info = sen.connection_kwargs
print(info)

# 监控端
print(sen.sentinels)
```

## redis-sentinel实现原理

### 三个定时监控任务
每隔10s，Sentinel会向master以及slave发送info 命令获取最新的拓扑结构
![2020-11-02-14-34-46](http://noback.upyun.com/2020-11-02-14-34-46.png)
这样的作用是 
首选 获取主从节点的信息 
其次 及时更新从节点 
最后 节点不可达或者故障转移后，可以通过info命令实时更新节点拓扑信息

每隔2s，每个Sentinel节点会向Redis数据节点的__sentinel__：hello频道上发送该Sentinel节点对于主节点的判断以及当前Sentinel节点的信息（如图9-27所示），同时每个Sentinel节点也会订阅该频道，来了解其他Sentinel节点以及它们对主节点的判断，所以这个定时任务可以完成以下两个工作：
首先 发现新的Sentinel节点
其次 交换主节点状态，达到一定数量时候，启动故障转移程序以及领导者选举

每隔1秒，每个Sentinel节点会向主节点、从节点、其余Sentinel节点发送一条ping命令做一次心跳检测，来确认这些节点当前是否可达

### 主观下线与客观下线

主观下线
每个Sentinel节点会每隔1秒对主节点、从节点、其他Sentinel节点发送ping命令做心跳检测，当这些节点超过down-after-milliseconds没有进行有效回复，Sentinel节点就会对该节点做失败判定，这个行为叫做主观下线。
另外 Sentinel 与 从节点在主观下线后，没有后续的故障转移
![2020-11-02-14-43-42](http://noback.upyun.com/2020-11-02-14-43-42.png)



客观下线
当Sentinel主观下线的节点是主节点时，该Sentinel节点会通过sentinel is-master-down-by-addr命令向其他Sentinel节点询问对主节点的判断，当超过`quorum`个数，Sentinel节点认为主节点确实有问题，这时该Sentinel节点会做出客观下线的决定，这样客观下线的含义是比较明显了，也就是大部分Sentinel节点都对主节点的下线做了同意的判定，那么这个判定就是客观的




## 总结

### 高可用读写分离架构
![2020-11-02-14-47-16](http://noback.upyun.com/2020-11-02-14-47-16.png)
写数据从master中进去， slave 复制master中的数据
当主节点出现问题的时候，从节点顶上去


### 哨兵构建脚本

master部署
```bash
#!/bin/bash

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
  echo "copy ./redis.conf >> ./redis-$1.conf" 
  `cp ./redis.conf ./redis-$1.conf` 
fi

sed -i -e  "/^port/s/.*/port ${Port}/"  \
    -e "/^daemonize/s/no/yes/"   \
    -e "/^logfile/s/.*/logfile \"${Port}.log\"/" \
    -e "/^dbfilename/s/.*/dbfilename \"dump-${Port}.rdb\"/"   \
    -e "/^dir/s@.*@dir /opt/soft/redis/data/@" \
    -e 's/^bind/#&/' \
    -e "/^protected-mode/s/yes/no/" \
    redis-${Port}.conf

echo "success"
```

从机部署
```bash
#!/bin/bash

if test -z "$1"
then 
  echo "no set config name"
  exit 1
else 
  echo "config the redis-$1.conf now>>>>>>>"
  Port="$1"
fi 


if test -f "./redis-$1.conf"
then
  echo "file exit"
else
  echo "copy ./redis.conf >> ./redis-$1.conf" 
  `cp ./redis.conf ./redis-$1.conf` 
fi

sed -i -e  "/^port/s/.*/port ${Port}/"  \
    -e "/^daemonize/s/no/yes/"   \
    -e "/^logfile/s/.*/logfile \"${Port}.log\"/" \
    -e "/^dbfilename/s/.*/dbfilename \"dump-${Port}.rdb\"/"   \
    -e "/^dir/s@.*@dir /opt/soft/redis/data/@" \
    -e '$a slaveof 127.0.0.1 6379' \
    -e 's/^bind/#&/' \
    -e "/^protected-mode/s/yes/no/" \
    redis-${Port}.conf

echo "success"
```

哨兵部署
```bash
#!/bin/bash

if test -z "$1"
then 
  echo "no set config name"
  exit 1
else 
  echo "config the redis-$1.conf now>>>>>>>"
  Port="$1"
fi 


if test -f "./redis-$1.conf"
then
  echo "file exit"
else
  echo "copy ./redis.conf >> ./redis-$1.conf" 
  `cp ./redis.conf ./redis-$1.conf` 
fi

sed -i -e  "/^port/s/.*/port ${Port}/"  \
    -e "/^daemonize/s/no/yes/"   \
    -e "/^logfile/s/.*/logfile \"${Port}.log\"/" \
    -e "/^dir/s@.*@dir /opt/soft/redis/data/@" \
    -e 's/^bind/#&/' \
    -e "/^protected-mode/s/yes/no/" \
    -e '$a sentinel monitor mymaster 127.0.0.1 6379 2'  \
    -e '$a sentinel down-after-milliseconds mymaster 30000 ' \
    -e '$a sentinel parallel-syncs mymaster 1  ' \
    -e '$a sentinel failover-timeout mymaster 180000'  \
    redis-${Port}.conf

echo "success"
```


bash