---
title: linux命令curl
date: 2020-12-09 14:00:24
categories: [linux,]  
tags: [linuxcc,linux]
---


<!-- more -->
# linux命令curl


## 常用方式

> get请求

```bash
curl http://127.0.0.1:8080/login?admin&passwd=12345678
```

> post请求

```bash
curl -d "user=admin&passwd=12345678" http://127.0.0.1:8080/login
```

> 指定头部

```bash
curl -H "Content-Type:application/json" -X POST -d '{"user": "admin", "passwd":"12345678"}' http://127.0.0.1:8000/login
```

> upyun云存储上传

需要操作员的账号密码
```bash
upyunadmin="hzj"
upyunpassword="tDsrDfFZRl4vjKn7TbpTc12EYAGLsErK"
bucket="http://v0.api.upyun.com/cewenyundownload/"
curl --user ${upyunadmin}:${upyunpassword} --upload-file /root/code/${dbname} ${bucket}dbsave/${dbname}
```
获取内容
```bash
curl --user ${upyunadmin}:${upyunpassword} ${bucket}dbsave/${dbname}
```

> 只打印头部信息

```bash
curl -I http://man.linuxde.net
curl --head http://www.baidu.com
```