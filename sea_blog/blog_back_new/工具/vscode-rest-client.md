---
title: vscode-rest-client
date: 2021-03-31 14:17:52
categories:  [Tool]
tags: [vscode,curl]
---


<!--more-->


# vscode-rest-client
vscode中一个调试API的插件，蛮好用的，可以用来生成
![](https://noback.upyun.com/2021-03-31-14-19-53.png!)
安装完插件之后，可以在文件中创建一个`.http`的文件，下面说几个特性

- 使用三个井号`###`分割一个接口
- 支持`cURL`和`RFC 2616`两种方式测试接口
- 支持转换成多种语言的request方法
- 支持转换成Curl方法
- 支持文档注释
- 支持自定义全局变量


```rest
@test_url = http://localhost:9600
@port = 8000
@url = localhost

### 入口文件测试
curl http://127.0.0.1:9600

### rust 入口文件测试
curl {{test_url}}

### rust 获取头部header 
curl {{test_url}}/api/auth/auth/ -u "admin:admin"

### rocket 获取cookie
curl {{test_url}}/api/cookie  -b name="hzjsea"

### rocket 
curl {{test_url}}/api/2

### 
curl {{test_url}}/api/content/

### just_fail  back 406
curl {{test_url}}/api/status

### body data
curl {{test_url}}/api/people

### form data
POST  http://localhost:8000/api/todo HTTP/1.1
Content-Type: application/x-www-form-urlencoded
type=true&description="hzjsea"

### authen Action
POST {{test_url}}/users/host/auth HTTP/1.1
Content-Type: application/json
{
    "action": "check_auth",
    "source_addr": "helloworld",
     "data": {
         "status": 200,
         "authenticator_source":[0,1,2,3,4,5,6,7,8,9,11,12,13,14,15,16]
      }
}
```