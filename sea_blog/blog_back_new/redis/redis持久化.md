---
title: redis持久化
categories:
  - redis
tags:
  - redis
abbrlink: dcfaa904
date: 2020-11-05 15:38:35
---

<!-- more -->

# 持久化
Redis支持RDB和AOF两种持久化机制，持久化功能有效地避免因进程退出造成的数据丢失问题，当下次重启时利用之前持久化的文件即可实现数据恢复。
现在主流的方式都是AOF持久化，RDB只作为了解


## RDB
RDB持久化是把当前进程数据生成快照保存到硬盘的过程，触发RDB持久化过程分为手动触发和自动触发

### 保存方式
RDB的保存方式有两种，一种是同步保存，另一种是异步保存
- save 同步保存，阻塞当前Redis服务器，直到RDB过程完成为止，对于内存比较大的实例会造成长时间阻塞，线上环境不建议使用
- bgsave 异步保存，Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短
bgsave只会保存执行那一刻之前的数据，异步保存执行阶段的数据不会被保存

save机制已经被弃用。

### 自动触发机制
除了执行命令手动触发之外，Redis内部还存在自动触发RDB的持久化机制，例如以下场景：
- 使用save相关配置，如“save m n”。表示m秒内数据集存在n次修改时，自动触发bgsave。
- 如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点，更多细节见6.3节介绍的复制原理。
- 执行debug reload命令重新加载Redis时，也会自动触发save操作。
- 默认情况下执行shutdown命令时，如果没有开启AOF持久化功能则自动执行bgsave

### bgsave执行流程
![2020-11-09-14-12-45](http://noback.upyun.com/2020-11-09-14-12-45.png)

- 首先redis会判断时候有子进程在执行 比如RDB或者AOF的子进程，如果存在bgsave命令直接返回
- 父进程执行fork操作创建子进程，fork操作过程中父进程会阻塞，通过`info stats`命令查看`latest_fork_usec`选项，可以获取最近一个fork操作的耗时，单位为微秒
- 父进程fork完成后，bgsave命令返回“Background saving started”信息并不再阻塞父进程，可以继续响应其他命令 `redis-cli info stats | grep latest_fork_usec`
- 子进程创建RDB文件，根据父进程内存生成临时快照文件，完成后对原有文件进行原子替换。执行`lastsave`命令可以获取最后一次生成RDB的时间，对应info统计的rdb_last_save_time选项 `redis-cli info | grep rdb_last_save_time`
- 进程发送信号给父进程表示完成，父进程更新统计信息，具体见info Persistence下的rdb_*相关选项

### RDB文件处理
RDB文件保存在dir配置指定的目录下
RDB的名字由dbfilename决定
```bash
grep "^dir" redis.conf
grep "^dbfilename"  redis.conf
```
可以通过执行`config set dir{newDir}`和`config set dbfilename{newFileName}`运行期动态执行，当下次运行时RDB文件会保存到新目录。

文件压缩
Redis默认采用LZF算法对生成的RDB文件做压缩处理，压缩后的文件远远小于内存大小，默认开启，可以通过参数`config set rdbcompression{yes|no}`动态修改。
可以使用Redis提供的`redis-check-dump`工具检测RDB文件并获取对应的错误报告。
脚本地址再`redis-package/src/redis-check-dump`


### RDB的优缺点
优点
- RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据快照。非常适用于备份，全量复制等场景。比如每6小时执行bgsave备份，并把RDB文件拷贝到远程机器或者文件系统中（如hdfs），用于灾难恢复。
- Redis加载RDB恢复数据远远快于AOF的方式

缺点
- RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运行都要执行fork操作创建子进程，属于重量级操作，频繁执行成本过高。
- RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题


## AOF持久化存储
AOF（append only file）持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的。AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式。理解掌握好AOF持久化机制对我们兼顾数据安全性和性能非常有帮助。
设置配置 ，默认不开启  
aof的文件名字由appendfilename决定
aof的文件保存的地址由dir配置决定
```bash
sed -i "/^appendonly/s/no/yes/" redis.conf
grep "appendfilename" redis.conf
```
![2020-11-09-14-47-45](http://noback.upyun.com/2020-11-09-14-47-45.png)

- 所有的写入命令会追加到aof_buf（缓冲区）中。
- AOF缓冲区根据对应的策略向硬盘做同步操作。
- 随着AOF文件越来越大，需要定期对AOF文件进行重写，达到压缩的目的。
- 当Redis服务器重启时，可以加载AOF文件进行数据恢复。


### AOF文件的存储格式
AOF会直接采用文本协议格式来存储 ,比如一条`set hello world`的命令，AOF会存储这样的形式`*3\r\n$3\r\nset\r\n$5\r\nhello\r\n$5\r\nworld\r\n`
转义之后内容为
```bash
[root@dev ~]# echo -e  "*3\r\n$3\r\nset\r\n$5\r\nhello\r\n$5\r\nworld\r\n"
*3

set

hello

world
```
- 文本协议具有很好的兼容性。
- 开启AOF后，所有写入命令都包含追加操作，直接采用协议格式，避免了二次处理开销。
- 文本协议具有可读性，方便直接修改和处理。

AOF为什么把命令追加到aof_buf中？Redis使用单线程响应命令，如果每次写AOF文件命令都直接追加到硬盘，那么性能完全取决于当前硬盘负载。先写入缓冲区aof_buf中，还有另一个好处，Redis可以提供多种缓冲区同步硬盘的策略，在性能和安全性方面做出平衡

### AOF文件的同步策略
![2020-11-09-14-53-59](http://noback.upyun.com/2020-11-09-14-53-59.png)
- 配置为always时，每次写入都要同步AOF文件，在一般的SATA硬盘上，Redis只能支持大约几百TPS写入，显然跟Redis高性能特性背道而驰，不建议配置。
- 配置为no，由于操作系统每次同步AOF文件的周期不可控，而且会加大每次同步硬盘的数据量，虽然提升了性能，但数据安全性无法保证。
- 配置为everysec，是建议的同步策略，也是默认配置，做到兼顾性能和数据安全性。理论上只有在系统突然宕机的情况下丢失1秒的数据。（严格来说最多丢失1秒数据是不准确的，5.3节会做具体介绍到。）

### AOF的重写机制
为了保证AOF文件的不会被无限的方法，AOF通常会在一段时间后，被Redis重写。主要重写的内容为以下几点
- 进程内已经超时的数据不再写入文件。
- 旧的AOF文件含有无效命令，如del key1、hdel key2、srem keys、set a111、set a222等。重写使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令。
- 多条写命令可以合并为一个，如：lpush list a、lpush list b、lpush list c可以转化为：lpush list a b c。为了防止单条命令过大造成客户端缓冲区溢出，对于list、set、hash、zset等类型操作，以64个元素为界拆分为多条。

重写机制的存在 一方面是降低了文件占用的空间， 另一方面则是可以更快的被redis加载

### AOF重写流程
和RDB一样，AOF也支持两种触发的形式
- 手动触发 直接调用`bgrewriteaof`命令
- 自动触发 根据`auto-aof-rewrite-min-size`和`auto-aof-rewrite-percentage`参数确定自动触发时机。

```bash
auto-aof-rewrite-min-size # 运行AOF重写时的文件最小体积，默认为64MB
auto-aof-rewrite-percentage #代表当前AOF文件空间（aof_current_size）和上一次重写后AOF文件空间（aof_base_size）的比值。
# 自动触发时间
自动触发时机 = aof_current_size > auto-aof-rewrite-min-size &&（aof_current_size-aof_base_size）/aof_base_size>=auto-aof-rewrite-percentage
```
其中aof_current_size 和aof_base_size可以在info Persistence统计信息中查看。
```bash
127.0.0.1:6379> info Persistence
# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1604903649
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:6742016
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0
```

> 图解AOF重写流程

![2020-11-09-15-13-04](http://noback.upyun.com/2020-11-09-15-13-04.png)

1) 执行AOF重写请求，如果进程中正在执行AOF重写，请求不执行并返回如下响应
```bash
ERR Background append only file rewriting already in progress
```
1) 如果当前进程正在执行bgsave操作，当前重写命令延迟到上一个结束之后再执行，则返回内容为
```bash
Background append only file rewriting scheduled
```
2) 父进程执行fork创建子进程，开销等同于bgsave过程。
3) 主进程fork操作完成后，继续响应其他命令。所有修改命令依然写入AOF缓冲区并根据appendfsync策略同步到硬盘，保证原有AOF机制正确性。
3) 由于fork操作运用写时复制技术，子进程只能共享fork操作时的内存数据。由于父进程依然响应命令，Redis使用“AOF重写缓冲区”保存这部分新数据，防止新AOF文件生成期间丢失这部分数据。
4) 子进程根据内存快照，按照命令合并规则写入到新的AOF文件。每次批量写入硬盘数据量由配置aof-rewrite-incremental-fsync控制，默认为32MB，防止单次刷盘数据过多造成硬盘阻塞
5) 新AOF文件写入完成后，子进程发送信号给父进程，父进程更新统计信息，具体见info persistence下的aof_*相关统计
6) 父进程把AOF重写缓冲区的数据写入到新的AOF文件。
7) 使用新AOF文件替换老文件，完成AOF重写。


### 重启加载
使用新AOF文件替换老文件，完成AOF重写。
![2020-11-09-15-21-54](http://noback.upyun.com/2020-11-09-15-21-54.png) 

### 文件校验
加载损坏的AOF文件时，会拒绝启动，并打印如下的日志
```bash
# Bad file format reading the append only file: make a backup of your AOF file, 
then use ./redis-check-aof --fix <filename>
```
修复AOF文件过程 
1. 先备份损坏的文件
2. 尝试脚本修复`redis-check-aof--fix`
3. 尝试手动修复 先 `diff -u `查看， 然后人工补全


AOF文件可能存在结尾不完整的情况，比如机器突然掉电导致AOF尾部文件命令写入不全。Redis为我们提供了aof-load-truncated配置来兼容这种情况，默认开启。加载AOF时，当遇到此问题时会忽略并继续启动，同时打印如下警告日志：
```bash
# !!! Warning: short read while loading the AOF file !!!
# !!! Truncating the AOF at offset 397856725 !!!
# AOF loaded anyway because aof-load-truncated is enabled
```


## 问题定位和优化

正常情况下fork耗时应该是每GB消耗20毫秒左右，查看上一次fork操作的时间
```bash
redis-cli info stats | grep latest_fork_usec # 基本单位为微妙 进位为1000
```

优化
- 优先使用物理机或者高效支持fork操作的虚拟化技术，避免使用Xen。
- 控制Redis实例最大可用内存，fork耗时跟内存量成正比，线上建议每个Redis实例内存控制在10GB以内
- 合理配置Linux内存分配策略，避免物理内存不足导致fork失败，具体细节见12.1节“Linux配置优化”
- 降低fork操作的频率，如适度放宽AOF自动触发时机，避免不必要的全量复制等

子进程开销监控和优化
![2020-11-09-16-18-46](http://noback.upyun.com/2020-11-09-16-18-46.png)


