## 官方网址
http://www.h3c.com/cn/d_201001/662474_30005_0.htm

```bash
# 生成RSA密钥对。
<Sysname> system-view
[Sysname] public-key local create rsa
The range of public key size is (512 ~ 2048).
NOTES: If the key modulus is greater than 512,
       It will take a few minutes.
Input the bits in the modulus[default = 1024]:
Generating keys...
...............................................++++++
......++++++
.................++++++++
.....++++++++
.......                
# 以OpenSSH格式显示RSA主机公钥。
[Sysname]public-key local export rsa openssh
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgMSPi+xIkHkAo6E9LwLKWN+eN9EqW/6FIYEIlVKcpIa0
6IT4eSyq4OldeiZ9WorOiDqX3ROo4FmaTR/QCSK3C9whE1qz/4soVL1eHDdgzQCumKKsJCVaM5OdZ2sdNbEnhLucs8ZrfTgEkDB1hmbgzuDpWPokPfkQDD+8dC+hkFVV rsa-key
# 导出RSA密钥的主机公钥部分为OpenSSH格式的文件，文件名设为pub_ssh_file2。
[Sysname] public-key local export rsa openssh pub_ssh_file2
# 导出RSA密钥的主机公钥部分为SSH1格式的文件，文件名设为pub_ssh_file3。
[Sysname] public-key local export rsa ssh1 pub_ssh_file3 
```

https://blog.csdn.net/solitary_sen/article/details/94746544