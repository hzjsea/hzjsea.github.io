---
title: cdn学习
categories:
  - 技术补充
tags:
  - CDN
abbrlink: e7cb699c
date: 2020-05-27 15:05:13
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->



<!-- /code_chunk_output -->
<!-- more -->

CND流程图

![2020-05-27-22-28-13](http://noback.upyun.com/2020-05-27-22-28-13.png)



Q: 302调度的优缺点ii


https://www.cnblogs.com/flying1819/articles/8795423.html

用户 -> 请求 -> 服务器 -> keepalived+lvs -> marco -> bepo



用户   <<<<->>>>  边缘(....bopo ... ... )  <----> 中转(vista bepo .... )  <-----> 一级缓存 (....bepo ...) 
                                                   |
                                                   |
                                                   ↓
                                                   用户源

边缘的都是单线的，中转是多线的，中转的实现是为了保障用户源的运营商与用户源的运营商相互匹配


bepo是一个缓存组件，用来判断是否有缓存
用户请求CDN地址，CDN返回边缘节点IP给用户 ，用户请求这个IP  搜索边缘节点是否存在缓存内容，没有则去边缘的兄弟节点查找(集群里面) 如果没有 再去中转  如果中转没有 则去回源到用户源查找内容， 返回的同时，在边缘和中转上做缓存
<font color='red'>这里返回IP给用户以后，为什么会主动请求这个IP</font>

vista用来过滤一些本身就不需要缓存的内容


     post put
用户 -----> marco 边缘 -----> vista 中转   -----> vivi 一级缓存
            ↓                                      ↑
            ↓------------------------------------->↑



lists-cdn-edge CDN边缘机器列表
lists-cdn-v406 CDN中转机器列表
lists-cdn-v406403 CDN数据中心列表     -> ansible inventory文件
lists-cdn-down CDN下架机器列表  



高防IP
