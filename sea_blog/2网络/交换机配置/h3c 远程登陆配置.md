## h3c交换机配置SSH以及telnet登录验证
由于这些都是经常要用到的配置，如果只是简单的配置流程的话，这些可以直接复制粘贴就OK了



## ssh密码验证登录
```bash
ssh server enable 
#配置虚拟口
user-interface vty 0 4
authentication-mode scheme
user-role level-3
protocol inbound ssh

# 配置用户
local-user gsadmin class manage 
service-type ssh
password simple admin
authorization-attribute user-role level-3
authorization-attribute user-role network-admin
authorization-attribute user-role network-operator
quit
```


## ssh密钥验证登录
```bash
# 开启ssh服务
ssh server enable
# 导入密钥,分别分为pub密钥和upyun密钥
 public-key peer pub
  public-key-code begin	
  xxx
  public-key-code end
 peer-public-key end

public-key peer upyun
 public-key-code begin
		xxx
 public-key-code end
 peer-public-key end

# ssh公钥认证  admin --> pub    root--->upyun
ssh user root service-type stelnet authentication-type publickey assign publickey upyun

# 虚拟串口配置
user-interface vty 0 4
authentication-mode scheme
user-role level-3
protocol inbound ssh
quit

# 配置用户
local-user root class manage
service-type ssh
# 设置级别
authorization-attribute user-role level-3
authorization-attribute user-role network-admin
authorization-attribute user-role network-operator
quit
```



## telnet配置
```bash
# 开启telnet服务
telnet server enable

#配置虚拟口
user-interface vty 0 4
authentication-mode scheme
user-role level-3
protocol inbound telnet

# 配置相应的角色
local-user admin class manage
password simple admin
service-type telnet
authorization-attribute user-role level-3
authorization-attribute user-role network-admin
authorization-attribute user-role network-operator
quit
```