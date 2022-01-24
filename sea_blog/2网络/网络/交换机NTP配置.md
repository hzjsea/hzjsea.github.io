## NTP简介

网络时间协议 http://www.h3c.com/cn/d_201108/723461_30005_0.htm 

```bash
ntp-service enable
ntp-service unicast-server 172.16.1.150 source LoopBack0
ntp-service unicast-server 115.231.97.11 source LoopBack0
```

同步时间