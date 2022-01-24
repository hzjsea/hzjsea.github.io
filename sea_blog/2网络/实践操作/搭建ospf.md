## 背景

## 
使用5820交换机

\#Switch A
```bash
# 打开loopback
interface loopback 0
ip address 172.168.2.1/30

# 端口开放route模式
system-view
interface rang ten1/0/21 to ten 1/0/24
port link-mode route


# 配置端口
interface ten 1/0/21
ip address 192.168.21.2/30
interface ten 1/0/22
ip address 192.168.22.2/30
interface ten 1/0/23
ip address 192.168.23.2/30
interface ten 1/0/24
ip address 192.168.24.2/30

# 配置ospf
ospf 1
area 0
network 192.168.21.0 0.0.0.3
network 192.168.22.0 0.0.0.3
network 192.168.23.0 0.0.0.3
network 192.168.24.0 0.0.0.3
```

\#Switch B
```bash
# 打开loopback
interface loopback 0
ip address 172.168.1.1/30

# 端口开放route模式
system-view
interface rang ten1/0/21 to ten 1/0/24
port link-mode route


# 配置端口
interface ten 1/0/21
ip address 192.168.21.1/30
interface ten 1/0/22
ip address 192.168.22.1/30
interface ten 1/0/23
ip address 192.168.23.1/30
interface ten 1/0/24
ip address 192.168.24.1/30

# 配置ospf
ospf 1
area 0
network 192.168.21.0 0.0.0.3
network 192.168.22.0 0.0.0.3
network 192.168.23.0 0.0.0.3
network 192.168.24.0 0.0.0.3
```


## 出现问题
#### 1.ospf router-id选择的问题
如果ospf router-id没有选择正确会怎么样

如何修改ospf的router-id
首先ospf在没有指定router-id的情况下会去查看loopback的接口
如果有多个loopback，则根据ip的大小来规定
规定结束之后，需要重启ospf
<h3c>reset ospf ospf-id process


如果选取了其他地址作为router-id
可以手动设置loopback的地址为ospf的router-id
interface loopback 0
ip address xxxx.xxxx.xxxx.xxxx xxx
ospf 1 router-id xxxx