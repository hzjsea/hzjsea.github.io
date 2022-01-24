---
title: Python爬YouTube视频
categories:
  - python
tags:
  - python
abbrlink: b3a48f7b
date: 2019-12-29 20:39:12
---
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [youtube视频爬虫](#youtube视频爬虫)
  - [缘由](#缘由)
  - [使用you-get](#使用you-get)
  - [编写脚本](#编写脚本)
  - [获取uid](#获取uid)
  - [整理循环下载](#整理循环下载)
  - [源码](#源码)

<!-- /code_chunk_output -->
<!--more-->
# youtube视频爬虫

## 缘由
最近想学习nginx，无奈没有好的视频，网上的教程也是零零散散的，慕课网 51CTO 淘宝上又买不到太好的教程，主要还是贵，买来要是讲的烂还不能退QAQ

于是，在YouTobe上搜索了一下nginx的教程，我的天哪！油管简直就是一块宝地，好多付费的视频上面都能找到，长期混迹B站找视频的我，决定赶紧下下来，生怕待会就没了，那么问题来了，如何批量下载这些视频呢？

**当然拉，能不白嫖还不是不要白嫖，要是讲的好，赞助一下也很重要**

## 使用you-get

Github地址: https://github.com/soimort/you-get
不得不说，光看他的star，又是一块宝藏QAQ

## 编写脚本
既然有了下载器，接下来就是使用脚本批量下载了。
you-get是根据油管的视频网页地址去下载视频的，那么我们需要记录这一组视频列表中的url地址。
https://www.youtube.com/watch?v=Do1Opl6DbvM&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD
https://www.youtube.com/watch?v=wZAf-aE2KoI&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD
https://www.youtube.com/watch?v=ULcqKLwf6fE&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD
如上是三个测试地址，很不幸的是他们之间存在规律，但唯一发生变化的v=uid 其中的id是一串随机字符，但当我们不断刷新当前页面时会发现，uid并没有改变，也就是说在他们的数据库中存在一个key-value的值，每一个视频对一个随机uid,并且当这相对的关系生成之后，随机uid不会再发生改变。

## 获取uid
既然不会发生改变，那么我们就用爬虫来获取这些uid把
**注意**

![](http://noback.upyun.com/youtobe%E7%88%AC%E8%99%AB.png)
像这样的页面虽然在他的右侧出现了列表，但这样的情况在当我们使用爬虫+正则的时，会被下方的视频连接干扰，因此最好先进入播放列表后再爬取
![](http://noback.upyun.com/190909youtobe%E7%88%AC%E8%99%AB2.png)
像当前页面这样，我们所获取出来的uid就相对于完整一些
![](http://noback.upyun.com/190909youtobe%E7%88%AC%E8%99%AB2.png)

## 整理循环下载
OK,有了uid再加上之前我们发现的不变的元素，那么一条条完整的视频链接就有了


## 源码
```python
#!/usr/bin/python3
from you_get import common as you_get
import sys
import os
import time
import requests
import re

static_url = "https://www.youtube.com{0}list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD"
file_path = "D:\\video\youtobe_nginx_mooc"
# file_path = "F:\\video\youtobe_nginx_mooc"


resp = requests.get("https://www.youtube.com/playlist?list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD")
pattern = "\/watch\?v=[0-9a-zA-Z_-]{11}&"
video_url_lists = re.findall(pattern,resp.text)
# 去重
video_url_lists = list(set(video_url_lists))
print(len(video_url_lists))
for list in video_url_lists:
    format_url = static_url.format(list)
    print(format_url)
    print("start get video from YouTobe")
    sys.argv = ['you-get','-o',file_path,format_url]
    you_get.main()
    time.sleep(1)
print("download finished")

# print(type(resp)) ## 由于这是返回的类型是 <class 'requests.models.Response'>
# 但是在正则表达式中，要求是为str类型，因此如果我们使用pattern = re.findall('index' resp)时候就会报错
# 报错内容为TypeError: expected string or bytes-like object
# 修改 pattern = re.findall('index=',str(resp))
# 或者使用pattern = re.findall('index=',resp.text)

""""
https://www.bilibili.com/read/cv4360/
https://www.jianshu.com/p/e323cf85bd3d

"""

"""
https://www.youtube.com/watch?v=5i9Ce9vzGxE&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD&index=1
https://www.youtube.com/watch?v=Do1Opl6DbvM&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD&index=1
https://www.youtube.com/watch?v=R4p7xxd3BTo&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD&index=4
https://www.youtube.com/watch?v=Do1Opl6DbvM&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD&index=1
https://www.youtube.com/watch?v=Do1Opl6DbvM&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD&index=1
/watch?v=Do1Opl6DbvM&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD&index=1
https://www.youtube.com/watch?v=0JwSFgWUUQE&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD

https://www.youtube.com/watch?v=fvxAQsRy8Kk&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD&index=8
https://www.youtube.com/watch?v=R4p7xxd3BTo&list=PLAyxoOmo7O7eoz-Wkii_bwtBEbPwCWQzD&index=4
"""
```