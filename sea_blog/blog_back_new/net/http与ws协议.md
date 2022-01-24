---
title: http与ws协议
date: 2021-01-05 15:23:13
categories: [网络]  
tags: [web,网络,协议]
---


<!-- more -->

# http与ws协议

> 看前须知
> http是一种使用在应用层上的协议,ws即websocket协议是基于http连接后替换替换成websocket协议才实现的
> http协议基于tcp/ip协议族才能实现,http必须在握手之后才能完成连接
> http协议本身是无状态不持久类的协议，在后面的优化过程中 出现了keep-alive维持持久， cookie状态管理等内容


`tcp/ip协议族完成握手后 -> 执行http -> 替换成ws websocket`

http协议全名为(HyperText Transfer Protocol) 超文本传输协议(超文本转移协议) web以该协议为规范,完成从客户端到服务端等一系列的运作流程,而协议是指规则的规定.


## tcp/ip协议族
`tcp/ip`协议族并不只包括tcp协议与ip协议,它是互联网相关的各类协议的总称
![2021-01-05-16-02-23](http://noback.upyun.com/2021-01-05-16-02-23.png)
`TCP/IP`取自OSI五层模型中的前四层 也就是 应用层,传输层,网络层以及数据链路层
![2021-01-05-16-05-28](http://noback.upyun.com/2021-01-05-16-05-28.png)
- 用户在web页面发送一个http请求
- http请求达到传输层后,TCP协议把从应用层处收到的数据(HTTP请求报文)进行分割,并在各个报文上打上标记序号及端口号后转发给网络层
- 在网络层,增加座作为通信目的地的MAC地址后转发给链路层
- 链路层通过物理层线路达到另一端的物理层,再到达链路层,一步步拆分到应用层


## ip协议 arp协议 tcp协议 DNS协议
ip arp mac地址  路由选择
![2021-01-05-16-15-28](http://noback.upyun.com/2021-01-05-16-15-28.png)
![2021-01-05-16-18-07](http://noback.upyun.com/2021-01-05-16-18-07.png)
![2021-01-05-16-19-22](http://noback.upyun.com/2021-01-05-16-19-22.png)
![2021-01-05-16-20-05](http://noback.upyun.com/2021-01-05-16-20-05.png)
![2021-01-05-16-20-47](http://noback.upyun.com/2021-01-05-16-20-47.png)


## URL与URI
URI(统一资源表示符), URL(统一资源定位符) URI是URL的超集
![2021-01-05-16-27-04](http://noback.upyun.com/2021-01-05-16-27-04.png)

## http协议
无状态协议,HTTP是一种不保存状态,即无状态(stateless)协议.HTTP协议自身不对请求和响应之间的通信状态进行保存

使用HTTP协议，每当有新的请求发送时，就会有对应的新响应产生。协议本事并不保留之前的请求或响应报文的信息。这是为了处理大量事务，确保协议的课伸缩性，而特意把HTTP协议设计成如此简单的。但是也存在弊端，当业务处理变得棘手的情况多了,比如用户登录一家网站，即使他跳转到别的页面后，也需要保持登录状态，然而cookie就诞生了。有了cookie再用http协议通信，就可以管理状态了。

### http请求报文
![2021-01-05-16-31-00](http://noback.upyun.com/2021-01-05-16-31-00.png)
下面是客户端发送给HTTP服务端的请求报文中的内容
```
GET / HTTP/1.1
HOST: HACKR.JP
```
内容分别为 
- GET 请求访问服务器的类型
- /index.html 请求地址, 也叫做URI
- HTTP/1.1 http协议版本

请求报文是由 `请求方法` `请求URI` `协议版本` `可选的请求首部字段` `内容实体`构造而成的
![2021-01-05-16-34-33](http://noback.upyun.com/2021-01-05-16-34-33.png)


> 响应内容
```
HTTP /1.1 200 OK

Date：Tue，10 JUL 2016 10:50:20 GMT   
Content-length：398
Content-Type：text/html

<html>
...
```
- HTTP /1.1 协议版本 200 OK 响应状态码以及原因短语
- 响应首部字段 Date  创建响应的时间
- `<html>` 资源实体的主体(body)

### 请求访问服务器的类型

> GET获取资源

![2021-01-05-16-43-38](http://noback.upyun.com/2021-01-05-16-43-38.png)

> POST传输实体主体

![2021-01-05-16-44-02](http://noback.upyun.com/2021-01-05-16-44-02.png)

> PUT传输文件

![2021-01-05-16-44-15](http://noback.upyun.com/2021-01-05-16-44-15.png)

> HEAD获取报文首部

![2021-01-05-16-44-35](http://noback.upyun.com/2021-01-05-16-44-35.png)

> DELETE删除文件

![2021-01-05-16-45-25](http://noback.upyun.com/2021-01-05-16-45-25.png)

### 持久化
之前有讲过http本身是一种不持久的协议,每次建立http都需要进行TCP/IP通信,就要做三次握手以及四次断开的行为,一旦量高了,流量就会飙升,于是有了持久连接的方法, 特点是任意一段没有明确提出断开的请求,则保持TCP连接状态
持久化之前
![2021-01-05-16-48-07](http://noback.upyun.com/2021-01-05-16-48-07.png)
持久化以后
![2021-01-05-16-49-38](http://noback.upyun.com/2021-01-05-16-49-38.png)
在`http/1.1`中,所有的连接默认都是持久连接,持久连接的方法就是在首部字段中添加
```bash
Keep-Alive：300
Connection：keep-alive
```
比如
```bash
GET / HTTP/1.1
HOST: HACKR.JP
Keep-Alive：300
Connection：keep-alive
```

### 管线化
持久化连接使得多数请求以管线化方式发送,也就是不会堵塞请求.
![2021-01-05-16-55-11](http://noback.upyun.com/2021-01-05-16-55-11.png)

### cookie 有状态
为了解决http是一种无状态请求的问题,产生了cookie,使用cookie来对http的状态进行管理
cookie会根据服务端发送的像一个报文内的一个叫做`Set-Cookie`的首部字段信息,通知客户端保存Cookie,当下次客户端再次发送请求的时候,客户端会自动在请求报文中加入Cookie值后发送到服务端
服务端发现客户端发送过来的Cookie后,会去检查究竟是哪一个客户端发来的连接请求,然后对比服务器上的记录, 最后得到之前的状态信息

>流程
![2021-01-05-17-14-24](http://noback.upyun.com/2021-01-05-17-14-24.png)
![2021-01-05-17-14-31](http://noback.upyun.com/2021-01-05-17-14-31.png)

> 请求报文的变化

没有cookie信息时候的状态
```
GET /reader/ HTTP/1.1
Host: hackr.jp
*首部字段内没有Cookie的相关信息
```
服务端返回的响应报文中的cookie信息
```
HTTP/1.1 200 OK
Date: Thu, 12 Jul 2012 07:12:20 GMT
Server: Apache
＜Set-Cookie: sid=1342077140226724; path=/; expires=Wed,
10-Oct-12 07:12:20 GMT＞
Content-Type: text/plain; charset=UTF-8
```
客户端再次请求时,自带的cookie信息
```
GET /image/ HTTP/1.1
Host: hackr.jp
Cookie: sid=1342077140226724
```



### sesssion
pass



## HTTP报文
http报文组成
![2021-01-05-17-22-52](http://noback.upyun.com/2021-01-05-17-22-52.png)
请求报文与响应报文结构
![2021-01-05-17-23-55](http://noback.upyun.com/2021-01-05-17-23-55.png)
详细版本
![2021-01-05-17-24-04](http://noback.upyun.com/2021-01-05-17-24-04.png)


## HTTP报文首部

> 请求报文
![2021-01-05-17-27-20](http://noback.upyun.com/2021-01-05-17-27-20.png)


```
GET / HTTP/1.1

Host: hackr.jp
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:13.0) Gecko/20100101 Firefox/13.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*; q=0.8
Accept-Language: ja,en-us;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: keep-alive
If-Modified-Since: Fri, 31 Aug 2007 02:02:20 GMT
If-None-Match: "45bae1-16a-46d776ac"
Cache-Control: max-age=0
```

> 响应报文
![2021-01-06-11-02-24](http://noback.upyun.com/2021-01-06-11-02-24.png)


```
HTTP/1.1 304 Not Modified

Date: Thu, 07 Jun 2012 07:21:36 GMT
Server: Apache
Connection: close
Etag: "45bae1-16a-46d776ac"
```

> 首部字段的格式

首部字段的格式如下
```bash
首部字段名: 字段值
Content-Type: text/html
Keep-Alive: timeout=15, max=100
```
当 HTTP 报文首部中出现了两个或两个以上具有相同首部字段名时
会怎么样？这种情况在规范内尚未明确，根据浏览器内部处理逻辑
的不同，结果可能并不一致。有些浏览器会优先处理第一次出现的
首部字段，而有些则会优先处理最后出现的首部字段。

### 首部字段分类
- 通用首部字段
![2021-01-06-11-35-06](http://noback.upyun.com/2021-01-06-11-35-06.png)
- 请求首部字段
![2021-01-06-11-35-16](http://noback.upyun.com/2021-01-06-11-35-16.png)
- 响应首部字段
![2021-01-06-11-37-49](http://noback.upyun.com/2021-01-06-11-37-49.png)
- 实体首部字段
![2021-01-06-11-38-00](http://noback.upyun.com/2021-01-06-11-38-00.png)
