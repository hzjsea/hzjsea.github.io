---
title: CDN流程
categories:
  - 技术补充
tags:
  - CDN
abbrlink: 4533ed96
date: 2020-04-10 10:56:04
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [CDN流程](#cdn流程)
  - [CNAME与A记录](#cname与a记录)
    - [A记录](#a记录)
    - [CNAME记录](#cname记录)
  - [正向代理与反向代理](#正向代理与反向代理)
    - [正向代理](#正向代理)
    - [反向代理](#反向代理)
    - [缓存检测](#缓存检测)
  - [CDN基础流程](#cdn基础流程)
    - [CDN回源拉取资源的场景](#cdn回源拉取资源的场景)
    - [查看自己CNAME的解析流程](#查看自己cname的解析流程)
  - [别人的解释](#别人的解释)

<!-- /code_chunk_output -->

learn
<!-- more -->
# CDN流程
https://tool.chinaz.com/dns/?type=5&host=image.noback.top&ip=
## CNAME与A记录
了解一下域名解析中的A记录与CNAME记录
域名解析就是国际域名或者国内域名以及中文域名等域名申请后做的到IP地址的转换过程。IP地址是网路上标识您站点的数字地址，为了简单好记，采用域名来代替ip地址标识站点地址。
<font color='red'>域名的解析工作由DNS服务器完成</font> 
![2020-04-10-10-57-57](http://noback.upyun.com/2020-04-10-10-57-57.png)

### A记录
别名的一种用来记录域名到IP的过程，你可以有多个A记录，也就是多个二级域名记录来指定一个IP
![2020-04-10-11-02-26](http://noback.upyun.com/2020-04-10-11-02-26.png)
这是一个我的github-page的地址，做了A记录

### CNAME记录
别名的一种用来记录域名到域名的过程
![2020-04-10-11-03-52](http://noback.upyun.com/2020-04-10-11-03-52.png)
这里我用作了CNAME到我上一个A记录那里,我这里作为博客记录所以用了blog.noback.top作为记录
如果是作为图片服务器，你可以CNAME成Image.noback.top作为记录


## 正向代理与反向代理
https://segmentfault.com/a/1190000018262215


<font color='red'>
正向代理代理的对象是客户端，比如翻墙
反向代理代理的对象是服务端，比如内网穿透

</font>



出现这个概念，基本和Nginx跑不掉了
Nginx的功能 
1. 代理服务器缓存
2. 正反向代理
3. http服务器
4. 负载均衡


### 正向代理
类似于翻墙，客户和代理服务器在在一端，当客户向后端服务器发送信息的时候，会先交由代理服务器后转发至互联网，并将内容传递给后端服务器，后端服务器接收后响应内容再交由代理服务器转发给客户。客户端必须要进行一些特别的设置才能使用正向代理。
![2020-04-10-11-13-35](http://noback.upyun.com/2020-04-10-11-13-35.png)
你可以理解为客户本身无法访问客户端，无论是墙内墙外还是内网外网，本身是无法访问的，但是通过代理服务器就可以做到。

### 反向代理
反向代理是透明的，他不会向用户展示后端真实的服务器，而是想用户展示了一个反向代理服务器，当用户需要获取数据的时候，他只能向反向代理服务器发送一个请求，代理服务器手动请求后，负载均衡，缓存检测等等操作，反应到真实服务器上去，后端服务器再返回内容通过代理服务器转发到用户身上
![2020-04-10-11-20-42](http://noback.upyun.com/2020-04-10-11-20-42.png)
一般用反向代理做负载均衡的时候，都会暴露两台机器做NGINX+keepalive+lvs做公网机器，同时也有内网的IP  用户访问公网，代理服务器转发流量到只有内网IP的服务器集群 再到内网机器做响应这样很大程度上的保护了内网机器的安全性

### 缓存检测
只要是代理服务器都有缓存检测这个内容，也就是无论是正向代理(直接请求真实服务器)还是反向代理(请求代理服务器，间接请求真实服务器)都要经过代理服务器，这个时候第一次请求会在代理服务上做一些缓存，当第二次再次请求的时候 回去查看这些内容，如果请求的有缓存在则直接返回，不请求真实服务器，如果没有缓存，则需要 <font color='red'>回源</font> 回到源点(真实服务器)上拿数据


## CDN基础流程
![2020-04-10-11-36-24](http://noback.upyun.com/2020-04-10-11-36-24.png)
![2020-04-10-11-39-53](http://noback.upyun.com/2020-04-10-11-39-53.png)


### CDN回源拉取资源的场景
当CDN节点没有缓存用户请求的内容时，会回源拉取资源。
当CDN节点上缓存的内容已过期时，会回源拉取资源。

### 查看自己CNAME的解析流程
dig 如下
```bash
➜  linux git:(master) ✗ dig image.noback.top

; <<>> DiG 9.10.6 <<>> image.noback.top
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62829
;; flags: qr rd ra; QUERY: 1, ANSWER: 11, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;image.noback.top.              IN      A

;; ANSWER SECTION:
image.noback.top.       600     IN      CNAME   hzjblog.b0.aicdn.com.
hzjblog.b0.aicdn.com.   300     IN      CNAME   nm.aicdn.com.
nm.aicdn.com.           72      IN      A       183.131.200.61
nm.aicdn.com.           72      IN      A       183.131.200.68
nm.aicdn.com.           72      IN      A       183.131.200.69
nm.aicdn.com.           72      IN      A       183.131.200.72
nm.aicdn.com.           72      IN      A       183.131.200.74
nm.aicdn.com.           72      IN      A       1.81.5.176
nm.aicdn.com.           72      IN      A       1.81.5.188
nm.aicdn.com.           72      IN      A       1.81.5.189
nm.aicdn.com.           72      IN      A       1.81.5.190
```
nslookup如下
```bash
➜  linux git:(master) ✗ nslookup image.noback.top
Server:         10.0.5.201
Address:        10.0.5.201#53

Non-authoritative answer:
image.noback.top        canonical name = hzjblog.b0.aicdn.com.
hzjblog.b0.aicdn.com    canonical name = nm.aicdn.com.
Name:   nm.aicdn.com
Address: 183.131.200.61
Name:   nm.aicdn.com
Address: 183.131.200.68
Name:   nm.aicdn.com
Address: 183.131.200.69
Name:   nm.aicdn.com
Address: 183.131.200.72
Name:   nm.aicdn.com
Address: 183.131.200.74
Name:   nm.aicdn.com
Address: 1.81.5.176
Name:   nm.aicdn.com
Address: 1.81.5.188
Name:   nm.aicdn.com
Address: 1.81.5.189
Name:   nm.aicdn.com
Address: 1.81.5.190
```


## 别人的解释
https://segmentfault.com/a/1190000010631404
![2020-04-10-11-43-58](http://noback.upyun.com/2020-04-10-11-43-58.png)