# ssh密钥远程登陆
首先机器装完的时候，如果你创建了root用户，那么你就可以用root用户+密码去登陆服务器
但是一旦机器多了，即使使用ansible也会很麻烦，频繁输入密码


## 配置步骤

1. 生成本地密钥
```bash
ssh-keygen 
```
会生成两个文件，一个id_rsa(密钥) 一个id_rsa.pub(公钥)
2. 上传公钥
有两种方法 
一种是自己复制id_rsa到远程服务器 会自动生成一个~/.ssh/authorized_keys, <font color='green'>但是这样容易出错，因为公钥默认为一行</font>
另外一种方法是使用命令传
```bash
ssh-copy-id root@192.168.20.1
# 会要求用密码登陆一次
# 他会直接自动生成~/.ssh/authorized_keys 
```

## ssh权限不足的问题
报错内容:
```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
Permissions 0644 for '/home/robin/.ssh/id_rsa' are too open.  
It is recommended that your private key files are NOT accessible by others.  
This private key will be ignored.  
bad permissions: ignore key: /home/robin/.ssh/id_rsa  
```

解决方法:
```bash
chmod 755 ~/.ssh/
chmod 600 ~/.ssh/id_rsa ~/.ssh/id_rsa.pub 
chmod 644 ~/.ssh/know_hosts
```