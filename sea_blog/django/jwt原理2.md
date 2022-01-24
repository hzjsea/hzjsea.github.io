---
title: jwt解密加密过程
categories:
  - jwt
tags:
  - jwt
abbrlink: e8b3a2c2
date: 2020-01-20 13:07:46
---

learn jwt
<!-- more -->

# jwt解密加密过程
https://zhuanlan.zhihu.com/p/70275218


jwt实例如下
```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
虽然看着长，但无非就是以下三部分组成
```bash
Header.Payload.Signature
```
在这个网站上，你可以修改这三块内容上面的东西动态的修改jwt中内容
https://jwt.io/
![2020-03-30-10-47-10](http://img.noback.top/2020-03-30-10-47-10.png)
在右边可以看到，
- `Header`部分我们存放的是 算法信息`alg`和加载方式`JWT`
- `Payload`中我们填写的内容是一些用户信息，用户可以在里面填写自己需要填写的内容,不可以存放任何的敏感信息，因为这个仅仅只是被Base64URL加密完成后的内容，是可以被直接解码获取的
- `SIGNATURE` 中存放签名，将前两段连接起来之后在用 进行加盐并用HS256算法或者其他算法进行加密后形成 

<font color='red'>签名生成过程中，加盐的步骤就是添加随机字符串，在drf中随机字符串来自于django中的配置文件</font>

生成方式

<font color='red'>前两段头部和载荷都是通过Base64URL进行编码的，后最后一段是将前两段连接在一起，然后在通过相应的加密算法（这里HS256）进行加密过后的字符串。</font>

## 解密过程

<font color='red'>说是解密 其实是对输入的账号密码重新加密的过程，然后与之前的token值进行对比</font>

因为最后一段的签名是由于前两端内容用`.`连接后并加密后进行算法计算得出，所以是不可逆的过程，
在解密的过程中，首先提取用户在前端输入的账号密码中的部分信息，因为 Payload中不允许出现密码等敏感内容，因此
用户名和密码基本不会同时出现在 Payload中，
- 然后jwt对用户信息进行加密生成 Payload，
- 对算法和加载方式进行Base64URL加密
- 再将两者连接在一起 加盐 然后进行HS256加密
- 对比之前生成的token值，如果完全匹配，则pass，否则返回401


## jwt使用流程
- 首先前端展示登录页面，用户在登录界面中输入账号密码，发出登录请求
- 服务器接收登录请求，检验用户是否存在于数据库中，如果存在则继续，不存在则返回错误
- 查询存在用户后，对密码进行hash加密，并且比较数据库中的hash密码是否配对，配对成功则表示登录成功
- 登录成功之后返回一个token值 token值由jwt生成，  jwt需要获取header payload 以及 加盐后生成一串token值返回给用户
- 当前用户每次进行权限操作的时候，都要带着token在header上进行请求
- 服务端拿到带着token的头部请求后，对其中的内容进行带盐解密，解密出paylaod后在比较数据库中的内容，匹配则表示成功


## 问题1 token存放在哪里
生成后的token 存放在哪里，不应该是存放在程序里面的,
其实他是存放在
- 服务端

<font color='red'>存放在服务端的哪个地方，不是每次都要拿出来对比验证一下吗</font>

- 客户端
    收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息Authorization字段里面

<font color='red'>这里是指每次访问内容，比如你登录之后，返回得到一个token，然后在你登录期间，你要做一些只有当前用户才有权限的操作，服务器肯定会做一个权限判断，这个时候就要用到存储在你本地的token了</font>

另外一种方式，放在HTTP头部里面去访问
使用 HTTP 头带上 Bearer 属性 + token就可以支持跨域操作。
### jwt的缺点
JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

## 问题2 如果做到一定时间内token不改变
由于jwt中的 Payload是用来存放用户信息的，只要不存放敏感信息
比如直接把用户名和密码暴露到payload中。。。
一般情况下，你可以随意的填写其他的内容
```bash
iss：Issuer，发行者
sub：Subject，主题
aud：Audience，观众
exp：Expiration time，过期时间
nbf：Not before
iat：Issued at，发行时间
jti：JWT ID
desc: do not decryption me
```
在这个里面他会存放一个过期时间

## Base64与Base64URL
前面提到，Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符+、/和=，在 URL 里面有特殊含义，所以要被替换掉：=被省略、+替换成-，/替换成_ 。这就是 Base64URL 算法。