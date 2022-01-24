---
title: 消息队列与nsq
date: 2021-04-14 10:33:35
categories:  [消息队列]
tags: [消息队列,nsq,rust-nsq]
---


<!--more-->


# 消息队列与nsq
用nsq之前先来搞定消息队列

> 小红是小明的姐姐。
小红希望小明多读书，常寻找好书给小明看，之前的方式是这样：小红问小明什么时候有空，把书给小明送去，并亲眼监督小明读完书才走。久而久之，两人都觉得麻烦。
后来的方式改成了：小红对小明说「我放到书架上的书你都要看」，然后小红每次发现不错的书都放到书架上，小明则看到书架上有书就拿下来看。
书架就是一个消息队列，小红是生产者，小明是消费者。
这带来的好处有：
1.小红想给小明书的时候，不必问小明什么时候有空，亲手把书交给他了，小红只把书放到书架上就行了。这样小红小明的时间都更自由。
2.小红相信小明的读书自觉和读书能力，不必亲眼观察小明的读书过程，小红只要做一个放书的动作，很节省时间。
3.当明天有另一个爱读书的小伙伴小强加入，小红仍旧只需要把书放到书架上，小明和小强从书架上取书即可（唔，姑且设定成多个人取一本书可以每人取走一本吧，可能是拷贝电子书或复印，暂不考虑版权问题）。
4.书架上的书放在那里，小明阅读速度快就早点看完，阅读速度慢就晚点看完，没关系，比起小红把书递给小明并监督小明读完的方式，小明的压力会小一些。
这就是消息队列的四大好处：
1.解耦
每个成员不必受其他成员影响，可以更独立自主，只通过一个简单的容器来联系。
小红甚至可以不知道从书架上取书的是谁，小明也可以不知道往书架上放书的人是谁，在他们眼里，都只有书架，没有对方。
毫无疑问，与一个简单的容器打交道，比与复杂的人打交道容易一万倍，小红小明可以自由自在地追求各自的人生。
2.提速
小红选择相信「把书放到书架上，别的我不问」，为自己节省了大量时间。
小红很忙，只能抽出五分钟时间，但这时间足够把书放到书架上了。
3.广播
小红只需要劳动一次，就可以让多个小伙伴有书可读，这大大地节省了她的时间，也让新的小伙伴的加入成本很低。
4.削峰
假设小明读书很慢，如果采用小红每给一本书都监督小明读完的方式，小明有压力，小红也不耐烦。
反正小红给书的频率也不稳定，如果今明两天连给了五本，之后隔三个月才又给一本，那小明只要在三个月内从书架上陆续取走五本书读完就行了，压力就不那么大了。
当然，使用消息队列也有其成本：
1.引入复杂度
毫无疑问，「书架」这东西是多出来的，需要地方放它，还需要防盗。
2.暂时的不一致性
假如妈妈问小红「小明最近读了什么书」，在以前的方式里，小红因为亲眼监督小明读完书了，可以底气十足地告诉妈妈，但新的方式里，小红回答妈妈之后会心想「小明应该会很快看完吧……」
这中间存在着一段「妈妈认为小明看了某书，而小明其实还没看」的时期，当然，小明最终的阅读状态与妈妈的认知会是一致的，这就是所谓的「最终一致性」。
那么，该使用消息队列的情况需要满足什么条件呢？
1.生产者不需要从消费者处获得反馈
引入消息队列之前的直接调用，其接口的返回值应该为空，这才让明明下层的动作还没做，上层却当成动作做完了继续往后走——即所谓异步——成为了可能。
小红放完书之后小明到底看了没有，小红根本不问，她默认他是看了，否则就只能用原来的方法监督到看完了。
2.容许短暂的不一致性
妈妈可能会发现「有时候据说小明看了某书，但事实上他还没看」，只要妈妈满意于「反正他最后看了就行」，异步处理就没问题。
如果妈妈对这情况不能容忍，对小红大发雷霆，小红也就不敢用书架方式了。
3.确实是用了有效果
即解耦、提速、广播、削峰这些方面的收益，超过放置书架、监控书架这些成本。
否则如果是盲目照搬，「听说老赵家买了书架，咱们家也买一个」，买回来却没什么用，只是让步骤变多了，还不如直接把书递给对方呢，那就不对了。


> 个人认为消息队列的主要特点是异步处理，主要目的是减少请求响应时间和解耦。所以主要的使用场景就是将比较耗时而且不需要即时（同步）返回结果的操作作为消息放入消息队列。同时由于使用了消息队列，只要保证消息格式不变，消息的发送方和接收方并不需要彼此联系，也不需要受对方的影响，即解耦和。使用场景的话，举个例子：假设用户在你的软件中注册，服务端收到用户的注册请求后，它会做这些操作：校验用户名等信息，如果没问题会在数据库中添加一个用户记录如果是用邮箱注册会给你发送一封注册成功的邮件，手机注册则会发送一条短信分析用户的个人信息，以便将来向他推荐一些志同道合的人，或向那些人推荐他发送给用户一个包含操作指南的系统通知等等……但是对于用户来说，注册功能实际只需要第一步，只要服务端将他的账户信息存到数据库中他便可以登录上去做他想做的事情了。至于其他的事情，非要在这一次请求中全部完成么？值得用户浪费时间等你处理这些对他来说无关紧要的事情么？所以实际当第一步做完后，服务端就可以把其他的操作放入对应的消息队列中然后马上返回用户结果，由消息队列异步的进行这些操作。或者还有一种情况，同时有大量用户注册你的软件，再高并发情况下注册请求开始出现一些问题，例如邮件接口承受不住，或是分析信息时的大量计算使cpu满载，这将会出现虽然用户数据记录很快的添加到数据库中了，但是却卡在发邮件或分析信息时的情况，导致请求的响应时间大幅增长，甚至出现超时，这就有点不划算了。面对这种情况一般也是将这些操作放入消息队列（生产者消费者模型），消息队列慢慢的进行处理，同时可以很快的完成注册请求，不会影响用户使用其他功能。所以在软件的正常功能开发中，并不需要去刻意的寻找消息队列的使用场景，而是当出现性能瓶颈时，去查看业务逻辑是否存在可以异步处理的耗时操作，如果存在的话便可以引入消息队列来解决。否则盲目的使用消息队列可能会增加维护和开发的成本却无法得到可观的性能提升，那就得不偿失

## 涉及的设计模式

发布者订阅者模式是行为型设计模式
![](https://noback.upyun.com/2021-04-14-10-39-39.png!)

## 消息队列选型
- kafka  搭建集群的时候用dns做多组机器负载均衡
- nsq 同上 缺点顺序不一致

## 消息队列与即时通信
消息的处理一般分为两种，一种是`消息队列`,另一种则是`即时通信`，根据消息是否需要通过中间层转发来定义这两者的关系
- 即时消息通讯，也就是说消息从发送者一端发出后立即就可以达到接收者一端;
- 延迟消息通讯，即消息从某一端发出后，首先进入一个容器进行临时存储，当达到某种条件后，再由这个容器发送给另一端。这个容器作为中间件转存消息，这层中间件就就是消息队列


## 消息队列的基本功能与扩展功能
根据不同的扩展功能，消息队列的软件也会有所不同，但基本g功能基本不会变成，这些是成为一个消息队列的条件之一

> 消息队列的基本功能

- 支持消息发生  接口实现
- 消息暂存
- 消息异步消费

> 消息队列的功能 

- 消息的顺序 正序 or 乱序
- 投递可靠性保证
- 消息持久化  消息刷在内存中 or 消息刷在磁盘上  单机 or 集群
- 支持不同消息模型
- 多实例集群功能
- 分布式环境下的负载均衡

### 消息队列的基础设计
为了实现消息队列的基础功能，即消息的传输，存储和消费，

需要从以下几个维度去进行设计：
- 通信协议
- 存储选择
- 消费关系维护

> 通信协议

消息既是信息的载体，消息发送者需要知道如何构造消息，消息接收者需要知道如何解析消息，它们需要按照一种统一的格式描述消息，这种统一的格式称之为`消息协议`。没有格式的消息是没有意义的。

1) Kafka的Producer、Broker和Consumer之间采用的是一套自行设计的基于TCP层的协议。Kafka的这套协议完全是为了Kafka自身的业务需求而定制的，而非要实现一套类似于Protocol Buffer的通用协议

2) AMQP规范和JMS规范 java那块的 

> 存储选择

消息队列常常保存在链表结构中，拥有权限的进程可以向消息队列中写入或读取消息。

- 内存
- 本地文件系统
- 分布式文件系统 ?
- 关系型数据库
- NoSQL数据库

从速度上内存显然是最快的，对于允许消息丢失，
- 消息堆积能力要求不高的场景(例如日志)，内存会是比较好的选择。
- 关系型数据库则是最简单的实现可靠存储的方案，很适合用在可靠性要求很高，最终一致性的场景(例如交易消息)。

> 消费关系维护

消息队列需要支持点对点和发布／订阅模式的消费模型， 消费端的消费进度也需要记录，典型的如消费端重连的处理，参考Kafka对每个Consumer提供一个偏移量的支持。

另外消息队列选择Pull还是Push模型进行实现也非常重要。在消费端，ActiveMQ使用PUSH模型，而Kafka使用PULL模型，两者各有利弊。对于PUSH，broker很难控制数据发送给不同消费者的速度，而PULL可以由消费者自己控制，但是PULL模型可能造成消费者在没有消息的情况下盲等，这种情况下可以通过long polling机制缓解。对于几乎每时每刻都有消息传递的流式系统，使用Pull模型更合适

### 消息队列的扩展功能

> 消息有序支持

> 投递可靠性

> 消息确认机制

消息确认机制可以给消息一致性提供支持，包括发送端的确认和消费端的确认，AMQP 协议本身使用的是事务机制进行消息确认，但是事务机制性能较差，并且容易发生阻塞。
Kafka应用的是ACK机制，RabbitMQ也设计了单独的消息确认机制。

> 消息发送和投递方式

![](https://noback.upyun.com/2021-04-14-11-08-50.png!)

> 异步消费的好处

![](https://noback.upyun.com/2021-04-14-11-41-11.png!)

> 应用解耦

![](https://noback.upyun.com/2021-04-14-11-41-44.png!)

> 流量削峰，限流




### 消息队列发送与投递的几种形式


> 点对点形式 不可重复消费
<div style='font-size:20px;color:red'>消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并且消费消息。
消息被消费以后，queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。</div>

![](https://noback.upyun.com/2021-04-14-11-12-21.png!)

> 发布者订阅者形式 可以重复消费

<div style='font-size:20px;color:red'>消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。</div>



![](https://noback.upyun.com/2021-04-14-11-14-21.png!)


> 两种相结合

<div style='font-size:20px;color:red'>发布订阅模式下，当发布者消息量很大时，显然单个订阅者的处理能力是不足的。实际上现实场景中是多个订阅者节点组成一个订阅组负载均衡消费topic消息即分组订阅，这样订阅者很容易实现消费能力线性扩展。可以看成是一个topic下有多个Queue，每个Queue是点对点的方式，Queue之间是发布订阅方式。</div>

![](https://noback.upyun.com/2021-04-14-11-17-30.png!)


### 流行消息队列
点对点消费模式与订阅者发布者消费模式一开始由JMS定义的，因此JMS来定义了一套规范，如上。但是RabbitMQ,kafka并没有遵循JMS规范。


> RabbitMQ

RabbitMQ实现了AQMP协议，AQMP协议定义了消息路由规则和方式。
<div style='font-size:20px;color:red'>生产端通过路由规则发送消息到不同queue，消费端根据queue名称消费消息</div>
RabbitMQ既支持内存队列也支持持久化队列，消费端为推模型，消费状态和订阅关系由服务端负责维护，消息消费完后立即删除，不保留历史消息。


> RabbitMQ 点对点 感觉这样做意义不大。。

![](https://noback.upyun.com/2021-04-14-11-22-19.png!)

> RabbitMQ 发布者订阅者 

![](https://noback.upyun.com/2021-04-14-11-23-44.png!)

> Kafka

Kafka只支持消息持久化，消费端为pull模型 <div style='font-size:20px;color:red'>消费状态和订阅关系由客户端端负责维护，消息消费完后不会立即删除，会保留历史消息。因此支持多订阅时，消息只会存储一份就可以了。但是可能产生重复消费的情况。</div>  解决办法是客户端处理重复消费的信息

![](https://noback.upyun.com/2021-04-14-11-24-09.png!)


## 链接
消息队列使用的四种场景介绍，有图有解析
https://zhuanlan.zhihu.com/p/55712984
消息队列设计精要
https://tech.meituan.com/2016/07/01/mq-design.html
什么是消息队列
https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247485080&idx=1&sn=f223feb9256727bde4387d918519766b&chksm=ebd74799dca0ce8fa46223a33042a79fc16ae6ac246cb8f07e63a4a2bdce33d8c6dc74e8bd20&token=1755043505&lang=zh_CN###rd
消息队列的使用场景及优缺点
https://www.cnblogs.com/laojiao/p/9573016.html

## Nsq消息队列
Nsq消息队列是go写的一个消息队列

NSQ 官方地址
https://nsq.io/components/nsqd.html


### 本地单机搭建demo
```bash
# 下载地址 https://github.com/nsqio/nsq/releases
wget https://github.com/nsqio/nsq/releases/download/v1.2.0/nsq-1.2.0.linux-amd64.go1.12.9.tar.gz
tar -xvf nsq-1.2.0.linux-amd64.go1.12.9.tar.gz
cd nsq

# 编译安装
git clone https://github.com/nsqio/nsq
cd nsq
make
./test.sh
```


### docker-compose 搭建
```bash
# 安装docker
wget https://www.asfor.cn/download/sh/docker_install.sh && sudo bash docker_install.sh
# 安装docker-compose
sudo curl -L https://download.fastgit.org/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# 编写docker-compose.yml
version: "3"
services:
   ysdp-nsq-admin:
     image: nsqio/nsq:v1.1.0
     command: /nsqadmin -lookupd-http-address ysdp-nsq-nsqlookupd1:4161 -lookupd-http-address ysdp-nsq-nsqlookupd2:4261
     ports:
     - "4171:4171"
   ysdp-nsq-nsqd1:
     image: nsqio/nsq:v1.1.0
     hostname: ysdp-nsq-nsqd1
     command: /nsqd -tcp-address 0.0.0.0:4150 -data-path /usr/local/nsq/bin/data --http-address 0.0.0.0:4151 -lookupd-tcp-address ysdp-nsq-nsqlookupd1:4160 -lookupd-tcp-address ysdp-nsq-nsqlookupd2:4260 -broadcast-address ysdp-nsq-nsqd1
     volumes:
     - "./data1:/usr/local/nsq/bin/data"
     ports:
     - "4150:4150"
     - "4151:4151"
   ysdp-nsq-nsqd2:
     image: nsqio/nsq:v1.1.0
     hostname: ysdp-nsq-nsqd2
     command: /nsqd -tcp-address 0.0.0.0:4250 -data-path /usr/local/nsq/bin/data -http-address 0.0.0.0:4251 -lookupd-tcp-address ysdp-nsq-nsqlookupd1:4160 -lookupd-tcp-address ysdp-nsq-nsqlookupd2:4260 -broadcast-address=ysdp-nsq-nsqd2
     volumes:
     - "./data2:/usr/local/nsq/bin/data"
     ports:
     - "4250:4250"
     - "4251:4251"
   ysdp-nsq-nsqd3:
     image: nsqio/nsq:v1.1.0
     hostname: ysdp-nsq-nsqd3
     command: /nsqd -tcp-address 0.0.0.0:4350 -data-path /usr/local/nsq/bin/data --http-address 0.0.0.0:4351 -lookupd-tcp-address ysdp-nsq-nsqlookupd1:4160 -lookupd-tcp-address ysdp-nsq-nsqlookupd2:4260 -broadcast-address=ysdp-nsq-nsqd3
     volumes:
     - "./data3:/usr/local/nsq/bin/data"
     ports:
     - "4354:4350"
     - "4355:4351"
   ysdp-nsq-nsqlookupd1:
     image: nsqio/nsq:v1.1.0
     command: /nsqlookupd -http-address 0.0.0.0:4161 -tcp-address 0.0.0.0:4160 -broadcast-address ysdp-nsq-nsqlookupd1
     ports:
     - "4160:4160"
     - "4161:4161"
   ysdp-nsq-nsqlookupd2:
     image: nsqio/nsq:v1.1.0
     command: /nsqlookupd -http-address 0.0.0.0:4261 -tcp-address 0.0.0.0:4260 -broadcast-address ysdp-nsq-nsqlookupd2
     ports:
     - "4260:4260"
     - "4261:4261"

# 单节点
version: '3'
services:
  nsqlookupd:
    image: nsqio/nsq
    command: /nsqlookupd
    networks:
      - nsq-network
    hostname: nsqlookupd
    ports:
      - "4161:4161"
      - "4160:4160"
  nsqd:
    image: nsqio/nsq
    command: /nsqd --lookupd-tcp-address=nsqlookupd:4160 --broadcast-address=nsqd
    depends_on:
      - nsqlookupd
    hostname: nsqd
    networks:
      - nsq-network
    ports:
      - "4151:4151"
      - "4150:4150"
  nsqadmin:
    image: nsqio/nsq
    command: /nsqadmin --lookupd-http-address=nsqlookupd:4161
    depends_on:
      - nsqlookupd
    hostname: nsqadmin
    ports:
      - "4171:4171"
    networks:
      - nsq-network

networks:
  nsq-network:
    driver: bridge

# 运行compose
docker-compose up -d
```


<div style='font-size:20px;color:red'>
这里是有一个坑的，就是你在docker里面弄得话，他的127.0.0.1 其实是指向容器内部的，也就是说你不能设置 --broadcast-address=127.0.0.1  你应该指向nsqd这个容器名 后面出来的时候回指向  nsqd:4151  这个端口 
</br>
后面还需要在这个/etc/hosts中设置一下 将nsqd 映射到127.0.0.1上面
</div>

来讲一讲这个`--broadcast-address`参数，翻译过来就是个广播地址，nsq一共会开三个进程
- nsqd 也就是接收端，用来接收消息的
- nsqadmin 这个你可以看成是一个界面，其实也可以不用到他
- nsqlookupd 这个是个服务发现的工具， 当有多个nsqd的时候，上面再套一层nsqlookupd ，通过nsqlookupd 来统一进出口

那么好，


那么具体该怎么映射呢，先开启服务，然后登陆nsqadmin的地址，这里是`http://127.0.0.1:4171`,进入node
![](https://noback.upyun.com/2021-05-20-16-35-21.png!)
这里会显示一个广播的的地址。把这个复制下来做一个映射 就OK了。

### go 消费端

```golang
package main

import (
	"fmt"

	"github.com/nsqio/go-nsq"
)

type ConsumerT struct{}

func main() {
	InitConsumer("xx", "xx1", "localhost:4161") // 分别是topic channel  nsqlookupd_address
	select {}
}

func (*ConsumerT) HandleMessage(msg *nsq.Message) error {
	fmt.Println("receive", msg.NSQDAddress, "message:", string(msg.Body))
	return nil
}

func InitConsumer(topic string, channel string, address string) {
	cfg := nsq.NewConfig()
	c, err := nsq.NewConsumer(topic, channel, cfg)
	if err != nil {
		panic(err)
	}
	c.AddHandler(&ConsumerT{})

	if err := c.ConnectToNSQLookupd(address); err != nil {
		panic(err)
	}
}

```

### nsq几个重要的组件

![](https://noback.upyun.com/2021-04-14-16-40-06.png!)

- topic 主题
- channel  channel组与消费者相关，是消费者之间的负载均衡，channel在某种意义上来说是一个“队列”。每当一个发布者发送一条消息到一个topic，消息会被复制到所有消费者连接的channel上，消费者通过这个特殊的channel读取消息，实际上，在消费者第一次订阅时就会创建channel。
- messages 消费者可以选择结束消息，表明它们正在被正常处理，或者重新将他们排队待到后面再进行处理。每个消息包含传递尝试的次数，当消息传递超过一定的阀值次数时，我们应该放弃这些消息，或者作为额外消息进行处理。

- nsqd 它是一个单独的监听某个端口进来的消息的二进制程序。每个nsqd节点都独立运行，不共享任何状态。当一个节点启动时，它向一组nsqlookupd节点进行注册操作，并将保存在此节点上的topic和channel进行广播。

- nsqlookupd nsqlookupd服务器像consul或etcd那样工作，只是它被设计得没有协调和强一致性能力。每个nsqlookupd都作为nsqd节点注册信息的短暂数据存储区。消费者连接这些节点去检测需要从哪个nsqd节点上读取消息。

![](https://www.liwenzhou.com/images/Go/nsq/nsq5.gif)

<div style='font-size:20px;color:red'>
每一个topic都可以拥有多个channel来保证数据的分类，topic在首次发布的时候被创建
</div>
比如 创建一个新的topic 

```bash
# 直接创建一个topic=public的主题
curl -X POST  http://localhost:4151/topic/create?topic=public
# 指定主题发送消息 这里只能主题为publice 发送message消息 100 次
for i in {1..100};do curl -d "message" http://localhost:61548/pub?topic=public ;done
# 发送消息 并创建一个新的主题 这里发送的消息是Message 创建的主题是new_topic
curl -d "message" http://localhost:4151/pub?topic=new_topic 
```
![](https://noback.upyun.com/2021-05-20-11-25-53.png!)
![](https://noback.upyun.com/2021-05-20-11-35-09.png!)



> channel

通道，本身channel 只提供分类的作用，发送到topic的信息  会被共享到当前topic下的所有channel, 但是消费者获取的时候，只能获取对应topic+channel组合全符合时候的信息

```bash
# 创建一个topic
curl -X POST http://127.0.0.1:4151/topic/create?topic=channel_test
# 创建一个channel
curl -d 't1' 'http://127.0.0.1:4151/pub?topic=channel_test&channel=t1'
curl -d 't2' 'http://127.0.0.1:4151/pub?topic=channel_test&channel=t2'
curl -d 't3' 'http://127.0.0.1:4151/pub?topic=channel_test&channel=t3'
```

> nsqd

默认情况下 nsq的端口分类如下
nsqd负责接收，排队，分发消息的守护进程。

| port  | usage  |
|---|---|
|4150|tcp client|
|4151|http client|
|4152|https client|

> nsqlookupd

nsqlookupd管理网络拓扑信息。客户端通过nsqlookupd发现nsqd生产者和主题。并且接收nsqd广播的主题和通道。
因此nsqd在配置的时候需要指定直接的广播地址， 也就是  broadcast-address 

| port  | usage  |
|---|---|
|4160|tcp client|
|4161|http client|

> nsqadmin

默认端口是4171 nsqadmin就是一个后端的UI界面，可以方便的看到各类的信息

| port  | usage  |
|---|---|
| 4171   | http client  |


> 消费方式

消费机制，消费一个message ,并将消息写入文件
```bash
./nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=127.0.0.1:4161
```


### 关于channel

这里要仔细来说说channel，首先channel只起到分类的作用，但是发往topic的信息，只要是在同一topic下面，那么这些信息就会被分配到当前topic下的所有channel当中，消费者根据自己获得的channel分类名获取信息。
这里可以来做一个测验，首先是运行上面的容器，然后跑通golang程序

```bash
package main

import (
	"fmt"
	"time"

	"github.com/nsqio/go-nsq"
)

var sum int = 20
type ConsumerT struct{}

func main() {
	go InitConsumer("public", "xx", "localhost:4161", 0)
	go InitConsumer("public", "xx1", "localhost:4161", 0)
	//go InitConsumer("public", "xx1", "localhost:4161", 10)
	select {}
}


func (*ConsumerT) HandleMessage(msg *nsq.Message) error {
	//time.Sleep(10*time.Second)
	sum = sum+1
	fmt.Println("sum----", sum, "receive", msg.NSQDAddress, "message:", string(msg.Body))
	return nil
}

func InitConsumer(topic string, channel string, address string, tt time.Duration) {
	cfg := nsq.NewConfig()
	c, err := nsq.NewConsumer(topic, channel, cfg)
	if err != nil {
		panic(err)
	}
	c.AddHandler(&ConsumerT{})

	if err := c.ConnectToNSQLookupd(address); err != nil {
		panic(err)
	}
}
```
接下来是创建过个通道
```bash
➜  nsq curl -X POST http://127.0.0.1:4151/channel/create\?topic\=public\&channel\=xx1
➜  nsq curl -X POST http://127.0.0.1:4151/channel/create\?topic\=public\&channel\=xx2
➜  nsq curl -X POST http://127.0.0.1:4151/channel/create\?topic\=public\&channel\=xx3
➜  nsq curl -X POST http://127.0.0.1:4151/channel/create\?topic\=public\&channel\=xx4
➜  nsq curl -X POST http://127.0.0.1:4151/channel/create\?topic\=public\&channel\=xx5
➜  nsq curl -X POST http://127.0.0.1:4151/channel/create\?topic\=public\&channel\=xx6
➜  nsq curl -X POST http://127.0.0.1:4151/channel/create\?topic\=public\&channel\=xx7
➜  nsq curl -X POST http://127.0.0.1:4151/channel/create\?topic\=public\&channel\=xx8
```
然后像某一个topic+channel中发生一段信息,
```bash
➜  nsq for i in {1..100};do curl -d "channel_message" http://localhost:4151/pub\?topic\=public\&channel\=xx1 ;done
OKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOK%
➜  nsq for i in {1..100};do curl -d "channel_message" http://localhost:4151/pub\?topic\=public\&channel\=xx1 ;done
OKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOK%
➜  nsq for i in {1..100};do curl -d "channel_message" http://localhost:4151/pub\?topic\=public\&channel\=xx1 ;done
OKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOK%
➜  nsq for i in {1..100};do curl -d "channel_message" http://localhost:4151/pub\?topic\=public\&channel\=xx1 ;done
OKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOKOK%
➜  nsq for i in {1..100};do curl -d "channel_message" http://localhost:4151/pub\?topic\=public\&channel\=xx4;done
```

再来看看admin下面的内容
![](https://noback.upyun.com/2021-05-20-17-43-54.png!)
可以看到 `xx1 和 xx`的都被消费完了，但是`xx2~xx8的数据没有被消费，这是因为上面的golang程序并没有主动的去调用`


### nsq配置优化

> nsq数据默认是存放在内存的，这样机器一当down掉，内存中的信息就会被释放

此配置由 `--mem-queue-size=0`时候，所有的消息将会存储到磁盘。

### nsq特性


- 默认一开始消息不是持久化的
nsq采用的方式时内存+硬盘的模式，当内存到达一定程度时就会将数据持久化到硬盘
1、如果将 --mem-queue-size 设置为 0，所有的消息将会存储到磁盘。
2、但是即使服务器重启也会将当时在内存中的消息持久化

- 消息是没有顺序的
这一点很关键，由于nsq使用内存+磁盘的模式，而且还有requeue的操作，所以发送消息的顺序和接收的顺序可能不一样

- 官方不推荐使用客户端发消息
官方提供相应的客户端发送消息，但是HTTP可能更方便一些

- 没有复制
nsq节点相对独立，节点与节点之间没有复制或者集群的关系。

- 没有鉴权相关模块
当前release版本的nsq没有鉴权模块，只有版本v0.2.29+高于这个的才有

- 几个小点
topic名称有长度限制，命名建议用下划线连接；
消息体大小有限制；

优点：
1、部署极其方便，没有任何环境依赖，直接启动就行
2、轻量没有过多的配置参数，只需要简单的配置就可以直接使用
3、性能高
4、消息不存在丢失的情况

缺点：
1、消息无顺序
2、节点之间没有消息复制
3、没有鉴权

 
### 链接
官网
https://nsq.io/components/nsqd.html
https://www.cnblogs.com/xuwc/p/14039073.html
http://liangjf.top/2019/06/05/76.nsq%E9%9B%86%E7%BE%A4%E5%92%8C%E8%B8%A9%E5%9D%91/
我们是如何使用NSQ处理7500亿消息的
http://www.jfh.com/jfperiodical/article/1949?

https://www.cnblogs.com/xuwc/p/14039073.html
https://ld246.com/article/1590046690321
https://www.liwenzhou.com/posts/Go/go_nsq/
https://www.cnblogs.com/li-peng/p/11435083.html

kafka 消息队列
https://www.zhihu.com/question/56172498
https://www.cnblogs.com/qingyunzong/p/9004509.html

## rust-nsq

很多nsq的使用方法和他官方提供的指令是一样的，包括需要提供的一些参数，在测试前先开一个nsq的服务
```bash
# 单节点
version: '3'
services:
  nsqlookupd:
    image: nsqio/nsq
    command: /nsqlookupd
    networks:
      - nsq-network
    hostname: nsqlookupd
    ports:
      - "4161:4161"
      - "4160:4160"
  nsqd:
    image: nsqio/nsq
    command: /nsqd --lookupd-tcp-address=nsqlookupd:4160 --broadcast-address=nsqd
    depends_on:
      - nsqlookupd
    hostname: nsqd
    networks:
      - nsq-network
    ports:
      - "4151:4151"
      - "4150:4150"
  nsqadmin:
    image: nsqio/nsq
    command: /nsqadmin --lookupd-http-address=nsqlookupd:4161
    depends_on:
      - nsqlookupd
    hostname: nsqadmin
    ports:
      - "4171:4171"
    networks:
      - nsq-network

networks:
  nsq-network:
    driver: bridge

# 运行compose
docker-compose up -d
```

nsqlookupd 需要监听nsqd的信息，并把消息发送到nsqadmin上显示


### 发布消息 /pub
![](https://noback.upyun.com/2021-06-23-11-05-41.png!)

需要提供的参数

1. address nsqd地址 producer的地址
2. message 发布的消息
3. topic 目的主题

对应的rust代码如下

```rust
// 生成需要发布的消息
fn make_default() -> (Arc<NSQTopic>, NSQProducer, NSQConsumer) {
    let topic = NSQTopic::new("helloworld".to_string()).unwrap(); // topic 
    let channel = NSQChannel::new("test").unwrap(); // channel
    let producer = NSQProducerConfig::new("127.0.0.1:4150").build(); // nsqd 地址
    let consumer = NSQConsumerConfig::new(topic.clone(), channel) // nsqlookupd 注册nsqd地址
       .set_max_in_flight(1)
        .set_sources(NSQConsumerConfigSources::Daemons(vec![
            "127.0.0.1:4150".to_string()
        ]))
        .build();
    (topic, producer, consumer)
}

// 发布消息
async fn run_message_tests(
    topic: Arc<NSQTopic>,
    mut producer: NSQProducer,
    mut consumer: NSQConsumer,
) {
    let mut large = Vec::new();
    large.resize(1 * 1024, 0);
    // 发布消息
    producer.publish(&topic, large.clone()).unwrap();

    // 消费消息
    let message = consumer.consume_filtered().await.unwrap();
    println!("{:?}",message);
    message.finish();
}

// 异步发送
fn main() -> Result<(),Box<dyn std::error::Error>>{
    let rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async{
            let (topic, producer, consumer) = make_default();
            run_message_tests(topic, producer, consumer).await;
        }
    );
    Ok(())
}
```

### 发布多个消息 /mpub

![](https://noback.upyun.com/2021-06-23-11-10-44.png!)

需要提供的参数

1. address nsqd地址 producer的地址
2. message1 message2 多条消息
3. topic 目的主题

```rust
fn main(){
    producer
        .publish_multiple(&topic, vec![b"alice".to_vec(), b"bob".to_vec()])
        .unwrap();
    // 消费该消息
    let message = consumer.consume_filtered().await.unwrap();
    assert_eq!(message.attempt, 1);
    assert_eq!(message.body, b"alice".to_vec());
    message.finish()
}
```

完整代码
```rust
use rand::distributions::Alphanumeric;
use tokio;
use tokio_nsq::*;
use core::fmt;
use std::{assert_eq, sync::Arc};
use ::rand::{thread_rng, Rng};


fn main() -> Result<(),Box<dyn std::error::Error>>{
    let rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async{
            let (topic, producer, consumer) = make_default();
            run_message_tests(topic, producer, consumer).await;
        }
    );

    Ok(())
}

fn make_default() -> (Arc<NSQTopic>, NSQProducer, NSQConsumer) {
    // let topic = random_topic();
    let topic = NSQTopic::new("h".to_string()).unwrap();
    let channel = NSQChannel::new("test").unwrap();
    let producer = NSQProducerConfig::new("127.0.0.1:4150").build();
    let consumer = NSQConsumerConfig::new(topic.clone(), channel)
       .set_max_in_flight(1)
        .set_sources(NSQConsumerConfigSources::Daemons(vec![
            "127.0.0.1:4150".to_string()
        ]))
        .build();
    (topic, producer, consumer)
}


async fn run_message_tests(
    topic: Arc<NSQTopic>,
    mut producer: NSQProducer,
    mut consumer: NSQConsumer,
) {
    // Several basic PUB
    // cycle_messages(topic.clone(), &mut producer, &mut consumer).await;

    // Large PUB
    let mut large = Vec::new();
    large.resize(1 * 1024, 0);

    producer.publish(&topic, large.clone()).unwrap();

    let message = consumer.consume_filtered().await.unwrap();
    println!("{:?}",message);
    message.finish();

    // MPUB
    producer
        .publish_multiple(&topic, vec![b"alice".to_vec(), b"bob".to_vec()])
        .unwrap();

    // let message = consumer.consume_filtered().await.unwrap();
    // assert_eq!(message.attempt, 1);
    // assert_eq!(message.body, b"alice".to_vec());
    // message.finish();

    // let message = consumer.consume_filtered().await.unwrap();
    // assert_eq!(message.attempt, 1);
    // assert_eq!(message.body, b"bob".to_vec());
    // message.finish();

    // // DPUB
    // producer
    //     .publish_deferred(&topic, b"hello".to_vec(), 1)
    //     .unwrap();
    // assert_matches!(producer.consume().await.unwrap(), NSQEvent::Ok());

    // let message = consumer.consume_filtered().await.unwrap();
    // assert_eq!(message.attempt, 1);
    // assert_eq!(message.body, b"hello".to_vec());
    // message.finish();
}
```

更多案例， look here https://github.com/harporoeder/tokio-nsq/blob/master/tests/test_live.rs