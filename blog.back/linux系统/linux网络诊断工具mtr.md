MTR是Linux平台上一款非常好用的网络诊断工具，集成了traceroute、ping、nslookup的功能，用于诊断网络状态非常有用。下面请看简单介绍。

## 安装
```bash
yum install mtr -y
```
## 使用方法
```bash
mtr ip或域名
                                                                   My traceroute  [v0.85]
PG-VPN (0.0.0.0)                                                                                                                   Wed Oct  9 18:22:54 2019
Resolver: Received error response 2. (server failure)er of fields   quit
                                                                                                                   Packets               Pings
 Host                                                                                                            Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. 115.231.97.1                                                                                             0.0%    20    0.8   1.0   0.8   1.3   0.0
 2. 10.100.129.24                                                                                           0.0%    20    0.9   0.9   0.9   1.0   0.0
 3. 10.100.128.65                                                                                            0.0%    20    0.9   1.0   0.9   1.1   0.0
 4. 10.100.129.27                                                                                            0.0%    20    1.3   1.1   1.0   1.3   0.0
 5. 124.14.9.238                                                                                              0.0%    20    4.0   4.1   3.8   4.3   0.0
```


第一列(Host) : IP地址和域名 按n键可以切换IP和域名
第二列(Loss%): 丢包率
第三列(Snt): 设置每秒发送数据包的数量，默认是10 可以通过参数-c来指定
第四列(Last)： 最近一次的Ping值
第五六七列(AVG BEST WRST) : 分别是Ping的平均 最好 最差值
第八列(StDev): 标准偏差

#### 其他用法
```bash
$ mtr -h  #提供帮助命令
$ mtr -v  #显示mtr的版本信息
$ mtr -r  #已报告模式显示
$ mtr -s  #用来指定ping数据包的大小
$ mtr --no-dns  #不对IP地址做域名解析
$ mtr -a  #来设置发送数据包的IP地址 这个对一个主机由多个IP地址是有用的
$ mtr -i  #使用这个参数来设置ICMP返回之间的要求默认是1秒
$ mtr -4  #IPv4
$ mtr -6  #IPv6
```