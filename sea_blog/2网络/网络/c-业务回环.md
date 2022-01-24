## 业务环回

<font color="red">为什么要加入业务环回组?    </font>

通过将一台设备上的一个或多个以太网端口绑定成为一个逻辑上的组合，这样的组合就称为业务环 回组，而这些被绑定的端口就称为该业务环回组的成员端口。业务环回组主要用于将某些特定业务 的报文进行内部环回，从而实现业务报文的正常转发。此外，当业务环回组中存在多个成员端口时， 还能实现业务流量的负载分担 



## 业务环回支持的类型

Tunnel 用于支持单播隧道业务

Multicast tunnel 用于支持组播隧道业务

## 业务环回组对成员端口的要求

端口必须支持该业务环回组的业务类型

端口尚未加入任何业务环回组



## 业务环回配置

```bash
system-view
service-loopback group number type (multicast tunnel | tunnel) # 创建业务环回组，并指定其业务类型
interface interface-type interface-number # 进入以太网接口
port service-loopback group number # 将端口加入业务环回组
```



## 显示业务环回组

```bash
display service-loopback group 1
```





h3交换机要走gre的话，要走一下业务环回流量，跟底层有关





## 案例

```bash
system-view
service-loopback group 1 type tunnel # 创建业务环回组，指定类型为Tunnel类型
# 加入端口
interface ten 1/0/1
port service-loopback group 1
quit
interfce ten 1/0/2
port service-loopback group 1
quit
interface ten 1/0/3
port service-loopback group 1
quit
# 创建 GRE 模式的 Tunnel 接口 1，该接口将自动引用业务环回组1
interface tunnel 1 mode gre
quit
```

