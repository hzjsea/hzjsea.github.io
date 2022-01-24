---
title: curl
date: 2021-03-31 14:32:01
categories:  [Tool]
tags: [curl]
top: 1
---


<!--more-->


# curl

curl 是利用 URL 语法在命令行方式下工作的开源文件传输工具。它被广泛应用在 Unix、多种 Linux 发行版中，并且有 DOS 和 Win32、Win64 下的移植版本。在日常的开发和问题处理中，经常会使用 curl 命令来测试 http 接口

> 常用转换
https://curl.trillworks.com/#rust
vscode rest client 插件
postman转换  不推荐这个软件要收费

![](https://noback.upyun.com/2021-03-31-14-33-42.png!)

主要介绍curl使用到的几个参数，以及具体的使用方法，curl支持短写的key 但是为了阅读方便，建议使用长写的key

## 常用实例

### POST信息
```bash
curl --request POST \
  --url http://localhost:9600/users/host/auth \
  --header 'content-type: application/json' \
  --header 'user-agent: vscode-restclient' \
  --data '{"action": "check_auth","source_addr": "helloworld","data": {"status": 200,"authenticator_source":[0,1,2,3,4,5,6,7,8,9,11,12,13,14,15,16]}}'


```

### 设置cookie 
```bash
-b/--cookie <name=string/file>    cookie字符串或文件读取位置
curl --request GET address -b name="hzjsea"
curl --request PST address --cookie name="hzjsea"
```

### 设置authorization
```bash
-u/--user <user[:password]>      设置服务器的用户和密码
curl address -u "admin:admin"
curl address --user "admin:admin"
```

### GET后将内容保存到文件当中
```bash
curl address -O 文件绝对路径
curl address >  xx.response
```

### POST内容
```bash
curl --request POST --header 'content-type: application/json' -d @json 文件绝对路径 address
curl --request POST --header 'content-type: application/json' -d '{"name":"hzjsea"}' address
curl --request POST --header 'content-type: application/json' -d @xml 文件绝对路径
address
curl --request POST --header 'content-type: application/json' -d 'xml 内容' address
```

### 设置代理字符串
```bash
curl URL --user-agent "代理内容"
curl URL -A "代理内容"
```

### 限制带宽速度
```bash
curl URL --limit-rate 速度
```

### 只打印头部信息
```bash
curl -I address
curl --head address
```

### 指定DNS
```bash
# 不需要修改 / etc/hosts，curl 直接解析 ip 请求域名
# 将 http://example.com 或 https://example.com 请求指定域名解析的 IP 为 127.0.0.1
curl --resolve example.com:80:127.0.0.1 http://example.com/
curl --resolve example.com:443:127.0.0.1 https://example.com/
```