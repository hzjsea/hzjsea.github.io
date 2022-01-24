---
title: redis基础
categories:
  - redis
tags:
  - redis
abbrlink: 78b568c7
date: 2020-11-02 10:24:45
---

![2020-10-28-17-47-35](http://noback.upyun.com/2020-10-28-17-47-35.png)
<!-- more -->

# redis基础

## Redis通用指令
```r
# keys* 查看所有的键
10.0.43.102:6379> lpush mylist h e ll o
(integer) 4
10.0.43.102:6379> lpush yourlist x x x x
(integer) 4
10.0.43.102:6379> set hello world
OK
10.0.43.102:6379> set name hzj
OK 
10.0.43.102:6379> set age hzj
OK
10.0.43.102:6379> sadd mydict name hzj object value
(integer) 4
10.0.43.102:6379>
10.0.43.102:6379>
10.0.43.102:6379> smembers mydict
1) "value"
2) "name"
3) "hzj"
4) "object"
10.0.43.102:6379> keys *
KEYS will hang redis server, use SCAN instead.
Do you want to proceed? (y/n): y
Your Call!!
1) "mylist"
2) "name"
3) "myset"
4) "yourlist"
5) "hello"
6) "mydict"
7) "age"

# 查看键总数
10.0.43.102:6379> dbsize
(integer) 7
# 查看键是否存在
10.0.43.102:6379> exists mydict
(integer) 1
# 删除键
10.0.43.102:6379> del mydict mylist name myset
DEL will delete keys, it may cause high latency when the value is big.
Do you want to proceed? (y/n): y
Your Call!!
(integer) 4
# 键过期
expire keys seconds
# 查看键的数据类型结构
type key

```
dbsize命令会在计算键总数的时候不会遍历所有键，而是直接获取redis内置的键总数的变量，所以dbsize命令的时间复杂度是O(1)
而keys命令会遍历所有键，所以他的时间复杂度是O(n) 当Redis保存了大量键的时候，线上环境精禁止的时候，他会产生大量的消耗


## redis单线程结构
redis的单线程架构，就好比一根水管，这个水管类似于一个队列，redis使用单线程来处理命令，一条命令从客户端达到服务端不会立刻被执行，所有命令都会进入一个队列中，然后逐个被执行
也就是可以被理解用 多个客户端，多个用户像redis发送命令的时候，这些命令都被塞进了一个管道里面，但是这个管道只允许每次出去一个命令到redis内部被执行，也就是说其他的命令需要被等待，知道上一个命令执行结束之后，才被允许进入到redis内部

<font color='red'>单线程运行速度快的原因?</font>

- 纯内存访问，redis将所有数据都放在内存当中，内存的响应时间大概为100ns 不管担心内存被撑爆，一来是数据量没有这么大，他也无法作为持久化存储的数据库 二来是redis3.0后面加入了集群的概念，物理机多的是，内存大小在大多是情况下，不是一个问题
- 非阻塞I/O redis使用epoll作为I/Od多路妇幼技术的实现，再加上Redis自身的时间处理模型将epoll中的连接 读写 关闭都转换为时间，不在网络I/O上浪费过多的时间
- 单线程避免了线程切换和竞态产生的消耗



## String键值对存储
字符串类型的值实际可以使字符串 数字 甚至是二进制数据，但是值不能超过512MB
### 创建和获取键值对
```bash
10.0.43.102:6379> set age 12
OK
10.0.43.102:6379> get age   # 这里拿到的就是string
"12"
10.0.43.102:6379> get a # 这里如果没有a这个key的话，返回的就是nil 即空
(nil)
10.0.43.102:6379> getset age 100 # getset 修改新的值，并且返回旧值
"12"
10.0.43.102:6379> set age 12  # set 赋值或者修改新的值，返回状态
OK
10.0.43.102:6379> mset a 100 b 200 c 300
OK
10.0.43.102:6379> mget a b c  # mget 返回的是一组数组
1) "100"
2) "200"
3) "300"

```
![2020-05-18-11-35-48](http://noback.upyun.com/2020-05-18-11-35-48.png)

批量获取可以加快返回速度，多次get+网络时间 = 单次mget

#### 键值对的中的类型转换
上面可以看到，我们存进去的`{"age":"12"}`中12是一个string类型，也就是一个字符串类型，但是在redis中可以进行原子增量，在执行的过程中，12会被转换成int 类型，并添加后转换成字符串类型
```bash
10.0.43.102:6379> incr age # 这里进行原子递增，redis内部进行变量类型转换
(integer) 13
10.0.43.102:6379> get age # 再去看value值得时候，他依旧是字符串类型
"13"
10.0.43.102:6379> incrby age 100 # 这里可以根据不同的增量值来增加
(integer) 113
10.0.43.102:6379> decr age # 减少
(integer) 112
10.0.43.102:6379> decrby age 100 # 按量减少
(integer) 12
```
### 判断key是否存在
`exists` 可以用来判断是否存在一个key, 存在则返回1 不存在则返回0
```bash
10.0.43.102:6379> get xx
(nil)
10.0.43.102:6379> exists xx
(integer) 0
10.0.43.102:6379> get age
"12"
10.0.43.102:6379> exists age
(integer) 1
```
### 根据key删除对应键值对
`del` 可以根据key来删除键值对，并且在删除之前会发生交互保证安全，存在删除则返回1 不存在删除则返回0
```bash
10.0.43.102:6379> get age
"12"
10.0.43.102:6379> del age
DEL will delete keys, it may cause high latency when the value is big.
Do you want to proceed? (y/n): y
Your Call!!
(integer) 1
10.0.43.102:6379> get xx
(nil)
10.0.43.102:6379> del xx
DEL will delete keys, it may cause high latency when the value is big.
Do you want to proceed? (y/n): y
Your Call!!
(integer) 0
10.0.43.102:6379> get gg
(nil)
10.0.43.102:6379> del gg
DEL will delete keys, it may cause high latency when the value is big.
Do you want to proceed? (y/n): n
Canceled!
```
### 返回键值对中的value类型
TYPE命令返回存储在指定密钥处的值的类型
```bash
10.0.43.102:6379> set age 12
OK
10.0.43.102:6379> type ag
"none"
10.0.43.102:6379> type age
"string"
10.0.43.102:6379> incrby age 100
(integer) 112
10.0.43.102:6379> type age
"string"
```
### 设置秘钥生成时间

redis支持对某一个key-value创建生成时间，并且使用`ttl`获取该剩下的生存时间
```r
10.0.43.102:6379> set name hzj
OK
10.0.43.102:6379> expire name 200  # 创建ttl时间
(integer) 1
10.0.43.102:6379> ttl name # 获取当前ttl值
(integer) 195
10.0.43.102:6379> set hello world
OK
10.0.43.102:6379> expire hello 1
(integer) 1
10.0.43.102:6379> get hello
(nil)
10.0.43.102:6379> ttl hello
(integer) -2
10.0.43.102:6379> ttl age
(integer) -1
------------------------------
10.0.43.102:6379> set age 20 ex 100  # 在创建的时候就设置计数器 秒级
OK
10.0.43.102:6379> ttl age
(integer) 98
10.0.43.102:6379> set age 1 px 1  # 毫秒级别计时器
OK
10.0.43.102:6379> get age
(nil)

------------------------------ # 删除计数器，并且使得数据持久化 -1
10.0.43.102:6379> set age 200 ex 100
OK
10.0.43.102:6379>
10.0.43.102:6379> ttl age
(integer) 97
10.0.43.102:6379> get age
"200"
10.0.43.102:6379> persist age
(integer) 1
10.0.43.102:6379> ttl age
(integer) -1
10.0.43.102:6379> get age
"200"
```
使用ttl可以查看时间,
不存在的键 ttl值为-2  
存在的值但是没有计时的 ttl值为-1 
存在的值但是计时了的 ttl值为设置值
![2020-05-18-14-17-45](http://noback.upyun.com/2020-05-18-14-17-45.png)
### 根据键的状态更新和添加
xx 键必须存在，才可以设置成功
nx 键必须不存在，才可以设置成功
```r
10.0.43.102:6379> get xx
(nil)
10.0.43.102:6379> set xx 1 xx  
(nil)
10.0.43.102:6379> set xx 1
OK
10.0.43.102:6379> get xx
"1"
10.0.43.102:6379> set xx 1 xx
OK
10.0.43.102:6379> del xx
DEL will delete keys, it may cause high latency when the value is big.
Do you want to proceed? (y/n): y
Your Call!!
(integer) 1
10.0.43.102:6379> get xx
(nil)
10.0.43.102:6379> set xx 1 nx
OK
```

### 不常用合集

1. vlaue尾部追加值
```r
10.0.43.102:6379> get key
"value"
10.0.43.102:6379> append key hello
(integer) 10
10.0.43.102:6379> get key
"valuehello"
```
2. 获取value长度
```r
10.0.43.102:6379> strlen key
(integer) 10
10.0.43.102:6379> set chinese-word "你好世界"
OK
10.0.43.102:6379> strlen chinese-word
(integer) 12
```
每个中文字符占3个字节
3. 设置并返回原值
```r
10.0.43.102:6379> get key
"valuehello"
10.0.43.102:6379> getset key newkey
"valuehello"
10.0.43.102:6379> get key
"newkey"
4
```
4. 设定指定位置的字符
```r
10.0.43.102:6379> set redis pest
OK
10.0.43.102:6379> get redis
"pest"
10.0.43.102:6379> setrange redis 0 b
(integer) 4
10.0.43.102:6379> get redis
"best"
```
5. 获取部分字符串
```r
10.0.43.102:6379> get redis
"best"
10.0.43.102:6379> getrange redis 0 1
"be"
```

## Redis列表
Redis官方文档中有说道 列表在不同的语言中呈现了不同的展现形式，比如在python中列表是以`Linked List`的形式存在的 而array并不是以link的形式存在的 
列表（list）类型是用来存储多个有序的字符串
array存储的就是存储的时候 每个元素都会带上一个index 以及一个value 插入的时候，读index就ok了
Redis列表是通过链接列表实现的。这意味着即使您在列表中有数百万个元素，在列表的开头或结尾添加新元素的操作也会在固定时间内执行
也就是说插入的时间肯定大于头部与尾部添加新元素的时间
使用LPUSH命令将新元素添加到具有10个元素的列表的开头的速度与将元素添加到具有1000万个元素的列表的开头的速度相同
当快速访问大量元素的中间很重要时，可以使用另一种称为排序集的数据结构



<font color='red'>另外 redis中的列表也是有列表名的  列表名与value 成键值对</font>



### 左右互搏,列表添加元素
```r
10.0.43.102:6379> rpush hellolist A  # 第一次添加的时候会创建一个空列表
(integer) 1
10.0.43.102:6379> rpush hellolist B # 在列表的右边添加一个B元素
(integer) 2
10.0.43.102:6379> lpush hellolist C # 在列表的左边添加一个C元素
(integer) 3
10.0.43.102:6379> lrange hellolist 0 -1 # 根据返回获取列表内容
1) "C"
2) "A"
3) "B"
10.0.43.102:6379> lrange hellolist 0 0
1) "C"
```
`lrange`可以根据范围来获取列表中的内容，需要提供两个数据作为范围的边界，第一个表示起始位置，第二个表示结束的位置
其中第一个位置可以表示为
`lrange hellolist 0 0`
最后一个位置为列表长度-1  -1
`lrange hellolist 2 -1`
![2020-05-18-23-26-24](http://noback.upyun.com/2020-05-18-23-26-24.png)

### 批量添加元素
```r
10.0.43.102:6379> rpush hellolist   6 5 "foo bar"
(integer) 10
10.0.43.102:6379> lrange hellolist 0 -1
1) "6"
2) "5"
3) "foo bar"
# 插入
10.0.43.102:6379> rpush hellolist   6 5 "foo bar"
(integer) 3
10.0.43.102:6379> lrange hellolist 0 -1
1) "6"
2) "5"
3) "foo bar"
10.0.43.102:6379> linsert hellolist BEFORE 5 500 # 找到5这个元素之后，在他的前面插入500
(integer) 4
10.0.43.102:6379> lrange hellolist 0 -1
1) "6"
2) "500"
3) "5"
4) "foo bar"
10.0.43.102:6379> linsert hellolist AFTER 5 500 # 找到5这个元素之后，在他的后面插入500
(integer) 5
10.0.43.102:6379> lrange hellolist 0 -1
1) "6"
2) "500"
3) "5"
4) "500"
5) "foo bar"

# 查询获取列表指定索引下标的元素
10.0.43.102:6379> lindex hellolist -1
"foo bar"
# 获取列表长度
10.0.43.102:6379> llen hellolist
(integer) 5

# 修改指定索引下标的元素:
10.0.43.102:6379> lrange hellolist 0 -1
1) "6"
2) "500"
3) "5"
4) "500"
5) "foo bar"
10.0.43.102:6379> lset hellolist 0 newvalue
OK
10.0.43.102:6379> lrange hellolist 0 -1
1) "newvalue"
2) "500"
3) "5"
4) "500"
5) "foo bar"

```
### 列表删除元素并且显示删除的元素信息
```r
10.0.43.102:6379> rpush hello  a b c d e f g
(integer) 7
10.0.43.102:6379> lrange hello 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"
7) "g"
10.0.43.102:6379> rpop hello
"g"
10.0.43.102:6379> lpop hello
"a"
10.0.43.102:6379> lrange hello 0 -1
1) "b"
2) "c"
3) "d"
4) "e"
5) "f"
10.0.43.102:6379> rpop helloworld  # 当没有元素弹出的时候，就会返回nil
(nil)

# 删除指定个数的元素
10.0.43.102:6379> lpush alist a a a a a a a
(integer) 7
10.0.43.102:6379> lrem alist 5 a
(integer) 5
10.0.43.102:6379> lrange alist 0 -1
1) "a"
2) "a"
# 按照索引
```
### 列表限制
redis支持对列表做数量上的限制，也就是封顶列表。仅使用LTRIM命令记住最近的N个项目并丢弃所有最旧的项目。
的LTRIM命令类似于LRANGE，但是，而不是显示元件的规定的范围内将其设置在该范围作为新的列表值。给定范围之外的所有元素都将被删除。
```r
10.0.43.102:6379> lpush mylist h e ll o
(integer) 4
10.0.43.102:6379> lrange mylist 0 -1
1) "o"
2) "ll"
3) "e"
4) "h"
10.0.43.102:6379> ltrim mylist 0 2
OK
10.0.43.102:6379> lrange mylist 0 -1
1) "o"
2) "ll"
3) "e"
```

### 列表等待
列表类型中，如果我们请求一个空的列表，如果没有元素，则redis会返回一个`(nil)`的元素，但是在某些场景下并不适合这样的操作，
假设用户想要获取到一个固定的值，但是返回的是一个nil的值，需要等待其他的操作往redis中push值，如果这种需求是及时的，那么该用户则需要不间断的快速的`lrange mylist 0 0`来获取元素，很明显这样的设计会增加两边的压力
于是redis提供一个列表等待的功能，也就是用户可以对列表的请求中加一个计时器，在计时器结束前，一旦有值被push进来就直接将他pop出去
![2020-05-18-23-54-05](http://noback.upyun.com/2020-05-18-23-54-05.png)
如图使用blpop 或者brpop这里如果只是添加一个元素的话，没啥效果，但是如果一次性push多个内容的话，就有效果了
另外如果需要表明永久等待的话,时间可以设置为0 
格式: `blpop key [key ...] timeout` `brpop key [key ...] timeout`

```bash
·lpush+lpop=Stack（栈）

·lpush+rpop=Queue（队列）

·lpsh+ltrim=Capped Collection（有限集合）

·lpush+brpop=Message Queue（消息队列）
```

## Redis哈希
创建字典，key值映射一个字典
```r
# 创建一个redis哈希
hmset key field value
-------------
10.0.43.102:6379> hmset user:1000 username antirez birthyear 1977 verified 1
"OK"
10.0.43.102:6379> hget user:1000 username
"antirez"
10.0.43.102:6379> hgetall user:1000
1) "username"
   "antirez"
2) "birthyear"
   "1977"
3) "verified"
   "1"
10.0.43.102:6379> hincrby user:1000 birthyear 100
(integer) 2077

10.0.43.102:6379> hmset user username jzk
"OK"
10.0.43.102:6379> hmset info name hzj
"OK"
10.0.43.102:6379> hget info name
"hzj"
10.0.43.102:6379> hdel info name  # 删除某一对键值对值如果要删除info这个key 的话 直接用del info
(integer) 1
10.0.43.102:6379> hgetall info
(empty list or set)

# 计算fiel的个数
10.0.43.102:6379>  hmset user:1000 username antirez birthyear 1977 verified 1
"OK"
10.0.43.102:6379> hlen user:1000
(integer) 3
# 判断field是否存在
10.0.43.102:6379> hexists user:1000 username
(integer) 1
# 获取所有field值
10.0.43.102:6379> hkeys user:1000
1) "username"
2) "birthyear"
3) "verified"

# 计算vlaue的字符串长度
10.0.43.102:6379> hstrlen user:1000 username
(integer) 7
10.0.43.102:6379> hget user:1000 username
"antirez"


# 获取key下面所有的valuse值
10.0.43.102:6379> hmset user:1000 username antirez birthyear 1977 verified 1
"OK"
10.0.43.102:6379> hvals user:1000
1) "antirez"
2) "1977"
3) "1"

# 获取所有的field-value
10.0.43.102:6379> hgetall user:1000
1) "username"
   "antirez"
2) "birthyear"
   "1977"
3) "verified"
   "1"

# 计算key对应的field长度
10.0.43.102:6379> hgetall user:1000
1) "username"
   "antirez"
2) "birthyear"
   "1977"
3) "verified"
   "1"
10.0.43.102:6379> hstrlen user:1000 username
(integer) 7

```
![2020-05-19-00-05-08](http://noback.upyun.com/2020-05-19-00-05-08.png)
![2020-05-20-22-33-36](http://noback.upyun.com/2020-05-20-22-33-36.png)
![2020-05-20-22-35-08](http://noback.upyun.com/2020-05-20-22-35-08.png)

<font color='red'>序列化类型数据库与redis行存储的区别 序列化数据库需要在加入计入每条数据之前需要规定好字段，并且当介入值得时候，如果有一个字段为空，则必须设置为null 但是在redis中则无需这样的操作</font>


## Redis集合
Redis集是字符串的无序集合，Redis可以在每次调用时自由以任何顺序返回元素，因为与用户之间没有关于元素顺序的约定。
集合（set）类型也是用来保存多个的字符串元素，但和列表类型不一样的是，集合中不允许有重复元素，并且集合中的元素是无序的，不能通过索引下标获取元素。
```r
10.0.43.102:6379> sadd myset 1 2 3  # 就是一个集合就行了
(integer) 3
10.0.43.102:6379> smembers myset  # 输出集合中的成员
1) "1"
2) "2"
3) "3"
3) "3"
10.0.43.102:6379> srem myset 3  # 删除集合中的成员
SREM will delete items.
Do you want to proceed? (y/n): y
Your Call!!
(integer) 1
10.0.43.102:6379> scard myset # 计算集合中成员的个数
(integer) 2
# scard的时间复杂度为O（1），它不会遍历集合所有元素，而是直接用Redis内部的变量
10.0.43.102:6379> sismember myset 2 # 判断集合中是否存在某一个元素，存在则返回1 不存在则返回0
(integer) 1
10.0.43.102:6379> sismember myset dsds 
(integer) 0


# set集合中的内容是不重复的
10.0.43.102:6379> sadd myset 1 2 3 4 5 6 7 8 9 0 1
(integer) 8
10.0.43.102:6379> smembers myset  # 获取集合myset所有元素，并且返回结果是无序的
 1) "0"
 2) "1"
 3) "2"
 4) "3"
 5) "4"
 6) "5"
 7) "6"
 8) "7"
 9) "8"
10) "9"

# 随机从集合中返回指定个数的元素
10.0.43.102:6379> smembers myset  
1) "0"
2) "1"
3) "2"
4) "3"
5) "4"
6) "5"
7) "6"
8) "7"
9) "8"
10) "9"
10.0.43.102:6379> srandmember myset 2
1) "9"
2) "4"
10.0.43.102:6379> srandmember myset 3
1) "4"
2) "3"
3) "0"
10.0.43.102:6379> srandmember myset  # 默认输出1个
"3"

# 随机从集合中弹出一个元素  删除~
10.0.43.102:6379> sadd mymyset 1 2 3
(integer) 3
10.0.43.102:6379> smembers mymyset
1) "1"
2) "2"
3) "3"
10.0.43.102:6379> spop mymyset
SPOP will delete items.
Do you want to proceed? (y/n): y
Your Call!!
"1"
10.0.43.102:6379> smembers mymyset # 默认弹出一个
1) "2"
2) "3"
10.0.43.102:6379> spop mymyset 2  # 可以指定个数
SPOP will delete items.
Do you want to proceed? (y/n): y
Your Call!!
1) "2"
2) "3"
10.0.43.102:6379> smembers mymyset
(empty list or set)
```

### 集合间的操作
redis中的集合之间可以做交集补集这些操作
```bash
10.0.43.102:6379> sadd set1 it is dangerous dont touch it
(integer) 5
10.0.43.102:6379> sadd set2 it is safe , you can play it
(integer) 7
```
![2020-05-20-23-02-05](http://noback.upyun.com/2020-05-20-23-02-05.png)

<font color='red'>多个集合都适用，并不是只适用于两个集合</font>

求两个集合的交集
```bash
10.0.43.102:6379> sinter set2 set1
1) "is"
2) "it"
```
求两个集合的合集
```bash
10.0.43.102:6379> sunion set1 set2
 1) "is"
 2) "safe"
 3) "dont"
 4) "dangerous"
 5) "it"
 6) "can"
 7) "you"
 8) "touch"
 9) ","
10) "play"
```
求两个集合之间的差集
```bash
10.0.43.102:6379> sdiff set2 set1
1) ","
2) "you"
3) "can"
4) "safe"
5) "play"
```
将集合的求差 求和 交集存储起来
```bash
sinterstore destination key [key ...]
suionstore  destination key [key ...]
sdiffstore  destination key [key ...]

10.0.43.102:6379> sdiffstore newset  set1 set2
(integer) 3
10.0.43.102:6379> type newset
"set"
10.0.43.102:6379> smembers newset
1) "dangerous"
2) "touch"
3) "dont"
```


## Redis有序集合
上面的集合是无序的集合,它保留了集合不能有重复成员的特性，但不同的是，有序集合中的元素可以排序。因为是有序集合，就要设置一个序号来进行排序。与之前的无序集合不同的是，有序集合多了一个fileds用来存储index 就像一个班里面的成绩单，不同的人之间是不可以重复的，但是分数是可以重复的

创建一个有序集合
```bash
# zadd key score member [score member ...]
10.0.43.102:6379> zadd user:ranking 220 hzj 220 ccc 210 jjj
(integer) 3
```
可以看一下有序集合和无序集合的差别， `set`为无序集合， `zset`为有序集合
![2020-05-20-23-48-15](http://noback.upyun.com/2020-05-20-23-48-15.png)
有序集合相比集合提供了排序字段，但是也产生了代价，zadd的时间复杂度为O（log（n）），sadd的时间复杂度为O（1）

```bash
# 计算成员个数
10.0.43.102:6379> zcard user:ranking
(integer) 3

# 获取有序集合中某一个集合的score
10.0.43.102:6379> zscore user:ranking hzj
"220"

# 有序集合排序 排序的内容是根据score进行排序的
# 根据value获取排名id
10.0.43.102:6379> zrank user:ranking hzj
(integer) 2
# 反向查询排名id
10.0.43.102:6379> zrevrank user:ranking hzj
(integer) 0

10.0.43.102:6379> zadd rank 1 xx 2 yy 3 cc 4 dd 4 qq 5 jj
(integer) 6
10.0.43.102:6379> zrank rank xx   # 排名第一就是0开始
(integer) 0
10.0.43.102:6379> zrank rank yy
(integer) 1

# 删除成员
10.0.43.102:6379> zrem rank yy
(integer) 1
10.0.43.102:6379> zrem rank xx cc dd
(integer) 3

# 增加成员的分数
10.0.43.102:6379> zincrby rank 10 jj
"15"

# 指定返回获取内容
10.0.43.102:6379> zrange rank 0 2 WITHSCORES # 可以不加scores
1) 1 "xx"
2) 2 "yy"
3) 3 "cc"
10.0.43.102:6379> zrange rank 0 2
1) "xx"
2) "yy"
3) "cc"

# 根据指定分数返回获取内容
10.0.43.102:6379> zrangebyscore rank 2 5
1) "yy"
2) "cc"
3) "dd"
4) "qq"
5) "jj"
10.0.43.102:6379> zrangebyscore rank 2 5 WITHSCORES
1) 2 "yy"
2) 3 "cc"
3) 4 "dd"
4) 4 "qq"
5) 5 "jj"

# 获取指定分数的个数
10.0.43.102:6379> zcount rank 1 2
(integer) 2
```

### 集合之间的操作
创建两个有序集合
```bash
10.0.43.102:6379> zadd user:ranking:1 1 kris 91 mike 200 frank 220 tim 250 martin
(integer) 5
10.0.43.102:6379> zadd user:ranking:2 8 james 77 mike 625 martin 888 tom
(integer) 4
```

## 键管理
```bash
# 对键进行重命名  (需要确认)
10.0.43.102:6379> set name hzj
OK
10.0.43.102:6379> get name
"hzj"
10.0.43.102:6379> rename name nohzj
RENAME use DELETE command to overwrite exist key, it may cause high latency when the value is big.
Do you want to proceed? (y/n): y
Your Call!!
OK
10.0.43.102:6379> get name
(nil)
10.0.43.102:6379> get nohzj
"hzj"

# 对键进行重命名 (不需要确认)
10.0.43.102:6379> set name hzj
OK
10.0.43.102:6379> get name
"hzj"
10.0.43.102:6379> renamenx name xxhzj
(integer) 1
10.0.43.102:6379> get xxhzj
"hzj"
```
由于重命名键期间会执行del命令删除旧的键，如果键对应的值比较大，会存在阻塞Redis的可能性，这点不要忽视

```bash
# 随机返回一个键
10.0.43.102:6379> randomkey
"nohzj"
```


## Redis集合、有序集合、列表的区别
![2020-05-20-23-13-17](http://noback.upyun.com/2020-05-20-23-13-17.png)


