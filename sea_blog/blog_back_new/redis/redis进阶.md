---
title: redis进阶
categories:
  - redis
tags:
  - redis
abbrlink: 83f1687c
date: 2020-05-21 00:22:58
---

![2020-10-28-17-47-35](http://noback.upyun.com/2020-10-28-17-47-35.png)
<!-- more -->


# redis功能加强
redis 修改配置的两种方法
- 直接修改配置文件
- 通过命令行配置文件

## redis 慢查询分析

许多存储系统（例如MySQL）提供慢查询日志帮助开发和运维人员定位系统存在的慢操作。所谓慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阀值，就将这条命令的相关信息（例如：发生时间，耗时，命令的详细信息）记录下来，Redis也提供了类似的功能
![2020-05-21-00-24-56](http://noback.upyun.com/2020-05-21-00-24-56.png)
redis命令执行的生命周期
1. 发生命令
2. 多个命令进入redis管道进行排队等待redis执行
3. 命令执行
4. 返回结果

需要注意，慢查询只统计步骤3）的时间，所以没有慢查询并不代表客户端没有超时问题。

### 慢查询的两个配置参数

<font color='red'>修改redis的配置时候，如果使用的是命令行修改，为了保证redis的持久化，需要在最后使用`config rewrite`来保存所修改的配置，但是如果redis找不到conf配置文件的话，就会出现报错，之前使用的时候，用的是docker-redis ,启动的过程中，没有conf映射进去，后面就出现报错了</font>

慢查询需要两个参数，一个是`预设阈值` 另外一个是`慢查询记录存放的地址`
Redis提供了slowlog-log-slower-than和slowlog-max-len配置来解决这两个问题。
`slowlog-log-slower-than`就是那个预设阀值，它的单位是微秒（1秒=1000毫秒=1000000微秒），默认值是10000，假如执行了一条“很慢”的命令（例如keys*），如果它的执行时间超过了10000微秒，那么它将被记录在慢查询日志中。
`slowlog-max-len` 用来定义这个管道最多能存放多少条记录

```bash
# 使用方法
slowlog-log-slower-than < 0  对于任何命令都不会进行记录
slowlog-log-slower-than = 0  会记录所有的命令
slowlog-log-slower-than = 10000 会记录超时时间为10000的查询记录
```
每当查询超过了预设的阈值，redis就会记录这一条内容，同时redis在记录这些超过阈值的命令的时候都会也采用了管道的形式(列表) 
当慢查询日志列表已处于其最大长度时，最早插入的一个命令将从列表中移出，例如slowlog-max-len设置为5，当有第6条慢查询插入的话，那么队头的第一条数据就出列，第6条慢查询就会入列。

配置慢查询相关参数
```bash
config set slowlog-log-slower-than 20000 # 当查询时间超过20000微妙的时候，也就是20毫秒的时候 就会被记录
config set slowlog-max-len 1000 # 记录的数量上限，上限到1000的时候，并不会停止记录，而是会每记录一条抛弃最旧的一条
config rewrite # 重写记录到配置文件中
```
查看配置文件中是否有写入redis
![2020-05-21-10-57-35](http://noback.upyun.com/2020-05-21-10-57-35.png)



### 慢查询日志实践

<font color='red'>虽然慢查询日志是存放在Redis内存列表中的，但是Redis并没有暴露这个列表的键，而是通过一组命令来实现对慢查询日志的访问和管理</font>

因为测试环境里面要让redis自然的记录慢查询 这样的环境有点麻烦，所以可以尽量限制查询时间，比如当`slowlog-log-slower-than=0`的时候,redis会记录每一条记录


```bash
# 获取不存在的key 模拟慢查询命令
10.0.43.102:6379> get name hzj
(error) ERROR wrong number of arguments for 'get' command
10.0.43.102:6379> set name hzj
OK
10.0.43.102:6379> get name
"hzj"
10.0.43.102:6379> slowlog get  # 慢查询
1) Slow log id: 2
   Start at: 1590029952
   Running time(ms): 6
   Command: get name
   Client IP and port: 10.128.0.1:34842
   Client name:
2) Slow log id: 1
   Start at: 1590029950
   Running time(ms): 8
   Command: set name hzj
   Client IP and port: 10.128.0.1:34842
   Client name:
3) Slow log id: 0
   Start at: 1590029941
   Running time(ms): 15
   Command: config set slowlog-log-slower-than 0
   Client IP and port: 10.128.0.1:34842
   Client name:
# 根据数量获取最近的一条慢查询数据
10.0.43.102:6379> slowlog get 1   
1) Slow log id: 3
   Start at: 1590029960
   Running time(ms): 46
   Command: slowlog get
   Client IP and port: 10.128.0.1:34842
   Client name:
```
###  慢查询日志操作
```r
# 获取redis日志列表中慢查询的条数(长度) 
10.0.43.102:6379> slowlog LEN
1) Slow log id: 6

# 慢查询日志重置
10.0.43.102:6379> slowlog RESET
1) Slow log id: O
2) Slow log id: K
```

### 慢查询最佳实践
![2020-05-22-10-17-23](http://noback.upyun.com/2020-05-22-10-17-23.png)



## redis shell
redis支持shell操作，不需要进入redis实例再去执行，在安装包中自带了很多的命令行工具
比如 redis-cli  redis-server redis-sentinel等等

### redis-cli
- -r 将命令执行多次
- -i 每隔几秒执行一次命令(单位是秒)
- -x 表示从stdin中的数据作为redis-cli的最后一个参数
- -c 选线是连接Redis Cluster节点时需要使用的，可以防止moved和ask
- -a 如果配置了密码，就不需要手动输入auth命令了
- --slave 把当前客户端模拟成当前Redis节点的从节点，可以用来获取当前Redis节点的更新操作
- -h host 主机地址
- -p port 主机端口
- --rdb 请求Redis实例生成并发送RDB持久化文件，保存在本地。可使用它做持久化文件的定期备份
- --pipe 用于将命令封装成Redis通信协议定义的数据格式，批量发送给Redis执行
- --bigkeys选项使用scan命令对Redis的键进行采样，从中找到内存占用比较大的键值，这些键可能是系统的瓶颈。
- --eval 用于执行指定Lua脚本
- --latency  测试客户端到目标Redis的网络延迟
- --latency-history 分断的形式测试网络延时
- --latency-dist 以图标的形式输出统计信息
- --stat 实时查看Redis统计信息，key的数量等等 主要是看一些增量信息
- --no-raw 返回原始的格式(默认)
- --raw 格式化后的结果


![2020-11-06-11-10-48](http://noback.upyun.com/2020-11-06-11-10-48.png)
![2020-11-06-11-43-02](http://noback.upyun.com/2020-11-06-11-43-02.png)
### redis-benchmark
压测使用
- -c 代表客户端的并发数量 默认50
- -n 代表客户端请求的总量 默认100000
- -q 仅显示requests per second的信息
- -r 随机生成更多的key-value
- -P 表示每个请求的pipline(命令组)的数据量 默认为1
- -k 表示客户端是否使用keepalive 1使用 0不使用  默认为1
- --csv 将结果按照csv格式输出，便于后续处理

```bash
# 并发100 请求20000次 
/usr/local/redis-5.0.8/src/redis-benchmark -h 10.0.6.55 -p 6379 -c 100 -n 20000
# 随机插入键，-r会在key、count键上加一个12位的后缀  -r 1000 表示随机后面3位数字
/usr/local/redis-5.0.8/src/redis-benchmark -h 10.0.6.55 -p 6379 -c 100 -n 20000 -r 1000  
```

## 事务

事务就是一组动作的组合，要么一起执行，要么就不一起不执行。比如付款，在你点击确定付款之前，你都可以回退所有的操作，而从你点击商品到付款成功之前的一系列动作都可以被叫做事务

```r
10.0.43.102:6379> multi   # 开启事务
OK
10.0.43.102:6379> sadd user:name hellowrold xx                                                                  
QUEUED
10.0.43.102:6379> exec    # 结束事务并执行事务                                                           
1) "2"
10.0.43.102:6379>
```
在事务中所执行的每一条command都会被记录到一个事务组里面，并且返回一个`QUEUED`,执行`exec`所有被塞进事务组里面的command都会被发送到redis服务端上依次执行，但是他们依旧是在一个事务组里面的。
```r
10.0.43.102:6379> multi
OK
10.0.43.102:6379> set name hzj                                                                                                       <transaction>
QUEUED
10.0.43.102:6379> get name                                                                                                           <transaction>
QUEUED
10.0.43.102:6379> sadd user:hello hellowrold 1                                                                                       <transaction>
QUEUED
10.0.43.102:6379> discard                # 结束事务，并取消之前的事务组 不执行事务                                                                                              <transaction>
OK
10.0.43.102:6379> get name
"hzj"
```

`discard`可以停止事务的执行，并且取消之前塞进事务组里面的所有command

在事务中，如果命令错误，会提示报错，但是不会在之后执行事务时执行
```bash
10.0.43.102:6379> multi
OK
10.0.43.102:6379> ds                                                                                                                 <transaction>
(error) ERROR `ds` is not a valide Redis Command
10.0.43.102:6379> dsd                                                                                                                <transaction>
(error) ERROR `dsd` is not a valide Redis Command
10.0.43.102:6379> sd                                                                                                                 <transaction>
(error) ERROR `sd` is not a valide Redis Command
10.0.43.102:6379> sds                                                                                                                <transaction>
(error) ERROR `sds` is not a valide Redis Command
10.0.43.102:6379>                                                                                                                    <transaction>
10.0.43.102:6379> exec                                                                                                               <transaction>
(empty list or set)
```

## 事务运行时错误
![2020-05-22-10-54-58](http://noback.upyun.com/2020-05-22-10-54-58.png)
不同客户端在事务生成期间，对key进行了修改，导致事务执行的时候没有执行成功

![2020-05-22-10-58-56](http://noback.upyun.com/2020-05-22-10-58-56.png)
正常情况下应该是`javapython`



## redis for python

```py
# 加入依赖库
import  redis

# 创建客户端
client = redis.StrictRedis(host="10.0.43.102",port=6379)

# 创建key
client.set("key","python for redis")
client.get("key")


# 使用pipeline 打包命令组

# 创建一个不使用事务的pipeline组的对象
pipeline = client.pipeline(transaction=False)
# 往命令组里面添加内容
pipeline.set("hello","world")
pipeline.set("name","cy")
# 发送命令组 交给redis服务端处理
result = pipeline.execute()

```


## 客户端API
client list 命令能列出与redis服务端相连的所有客户端连接信息
```r
10.0.43.102:6379> client list
id=10 addr=127.0.0.1:56420 fd=8 name= age=404 idle=10 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping user=default
id=13 addr=10.128.0.1:60810 fd=9 name= age=7 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=32742 obl=0 oll=0 omem=0 events=r cmd=client user=default
```
其中
`id`客户端连接的唯一标识，这个id是随着Redis的连接自增的，重启Redis后会重置为0。
`addr` 客户端连接的ip和端口
`fd` socket的文件描述符，与lsof命令结果中的fs是同一个，如果fd=-1代表当前客户端不是外部客户端，而是Redis内部的伪装客户端。
`name`：客户端的名字，后面的client setName和client getName两个命令会对其进行说明。
`qbfu` 输入缓冲区中的使用量，
`qbuf-free` 输入缓冲区中的剩余量，动态调整
`输入缓冲区` Redis为每个客户端分配了输入缓冲区，它的作用是将客户端发送的命令临时保存，同时Redis从会输入缓冲区拉取命令并执行，输入缓冲区为客户端发送命令到Redis执行命令提供了缓冲功能，Redis没有提供相应的配置来规定每个缓冲区的大小，输入缓冲区会根据输入内容大小的不同动态调整，只是要求每个客户端缓冲区的大小不能超过1G，超过后客户端将被关闭

![2020-05-22-11-22-33](http://noback.upyun.com/2020-05-22-11-22-33.png)