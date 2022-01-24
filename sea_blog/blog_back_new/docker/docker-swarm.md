---
title: docker-swarm
categories:
  - docker
tags:
  - docker
abbrlink: 8f65
date: 2020-02-07 15:38:32
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker-swarm](#docker-swarm)
  - [docker-swarm使用](#docker-swarm使用)
  - [水平扩展](#水平扩展)
  - [水平扩展后容器的网络问题](#水平扩展后容器的网络问题)
  - [关于服务查看](#关于服务查看)

<!-- /code_chunk_output -->
<!-- more -->

# docker-swarm
![2020-02-07-16-06-40](http://noback.upyun.com/2020-02-07-16-06-40.png)

准备机器 三台centos7 IP分别为10.0.5.233 10.0.5.196 10.0.6.5
其中一台作为manager管理机，其他作为work随从机

```bash
# 声明管理机
[root@dev ~]# docker swarm init --advertise-addr=10.0.6.55
Swarm initialized: current node (yi6s9lbwc402zvv8384wiras7) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4tnasb86t2w9bppugns4v8f9vjuf635kkhfkt341lnd18sqf4c-9m7jfe2u493h3wpr24z7audiy 10.0.6.55:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions. 
```
会给出一段信息，其中node节点的token值，以及声明node节点的方法
```bash
# 声明node机
[root@196 hzj]# docker swarm join --token SWMTKN-1-4tnasb86t2w9bppugns4v8f9vjuf635kkhfkt341lnd18sqf4c-9m7jfe2u493h3wpr24z7audiy 10.0.6.55:2377
This node joined a swarm as a worker.
[root:~]#  docker swarm join --token SWMTKN-1-4tnasb86t2w9bppugns4v8f9vjuf635kkhfkt341lnd18sqf4c-9m7jfe2u493h3wpr24z7audiy 10.0.6.55:2377
This node joined a swarm as a worker.
```
可以在manager上查看node节点
```bash
[root@manager55 ~]# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
yi6s9lbwc402zvv8384wiras7 *   manager55           Ready               Active              Leader              19.03.5
wxq2hk8ozhv1x6fdm7wplra8t     work196             Ready               Active                                  19.03.5
zq7zgn8o9ozyqipw5rofub6m5     work233             Ready               Active                                  19.03.5
```


## docker-swarm使用
可以把docker-swarm开成一个集群的搭建，首先是建立一个service的集群，在service创建多个work进行工作
![2020-02-09-16-32-24](http://noback.upyun.com/2020-02-09-16-32-24.png)

```bash
# 创建服务 manager
docker service create --name demo busybox sh -c "while true;do sleep 3600;done"
# 服务查看
[root@manager55 ~]# docker service  ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
ob8p30vdkhnn        demo                replicated          1/1                 busybox:latest

# container查看
[root@manager55 ~]# docker service ps demo
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE         ERROR               PORTS
rbl6tfetw25m        demo.1              busybox:latest      manager55        Running             Running 3 hours ago
```

## 水平扩展
docker-swarm可以水平扩展容器
```bash
#水平扩展5个 
[root@manager55 ~]# docker service scale demo=5
demo scaled to 5
overall progress: 5 out of 5 tasks
1/5: running   [==================================================>]
2/5: running   [==================================================>]
3/5: running   [==================================================>]
4/5: running   [==================================================>]
5/5: running   [==================================================>]
verify: Waiting 4 seconds to verify that tasks are stable...
```
将5个service同配置部署到其他机器上面，分配的方式为平均分配


```bash
[root@manager55 ~]# docker service ps demo
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                 ERROR
                      PORTS
rbl6tfetw25m        demo.1              busybox:latest      manager55           Running             Running 3 hours ago

uc3tn3coc8q3        demo.2              busybox:latest      work233             Running             Running about a minute ago

dy1tza747g3w        demo.3              busybox:latest      work233             Running             Running 56 seconds ago

mqbt0myozide         \_ demo.3          busybox:latest      work196             Shutdown            Rejected about a minute ago   "No such imag
e: busybox:latest…"
mtb3x9mabokn         \_ demo.3          busybox:latest      work196             Shutdown            Rejected about a minute ago   "No such image: busybox:latest…"
yyq86fjpf0i4        demo.4              busybox:latest      manager55           Running             Running 50 seconds ago
mo12k2g7gx1s         \_ demo.4          busybox:latest      work196             Shutdown            Rejected about a minute ago   "No such image: busybox:latest…"
cqpsqtvfeo9y         \_ demo.4          busybox:latest      work196             Shutdown            Rejected about a minute ago   "No such image: busybox:latest…"
t8z0vqc0kow7         \_ demo.4          busybox:latest      work196             Shutdown            Rejected about a minute ago   "No such image: busybox:latest…"
fojfst6j5pa2        demo.5              busybox:latest      manager55           Running             Running about a minute ago 
```
这里在部署的过程中发生了一个错误，work196的机器因为没有配置好DNS，导致无法从网上拉去busybox的镜像文件，于是发生了这样的情况
首先work196收到请求去拉去busybox创建容器，但是没有DNS拉去不了，报错。根据自动分配的设置，将任务追加分配给其他work或者manage机器部署

部署之后，我们可以在work233的机器上查看起来的容器
```bash
[root@work233]:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
837720ea43f8        busybox:latest      "sh -c 'while true;d…"   17 minutes ago      Up 17 minutes                           demo.3.dy1tza747g3whbv2zuasgc6s5
69d08591e255        busybox:latest      "sh -c 'while true;d…"   17 minutes ago      Up 17 minutes                           demo.2.uc3tn3coc8q39izu3to58z9or 
```

<font color='red'>manager机器查看work机器工作状态</font>
```bash
[root@manager55 ~]# docker service  ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
ob8p30vdkhnn        demo                replicated          5/5                 busybox:latest
```
其中REPLICAS 5/5 前者表示当前一共运行多少了容器，后者表示一共复制了多少了容器

尝试关闭work233机器上的一台容器，查看工作状态
```bash
[root@work233 ~]:~# docker rm -f 837720ea43f8
837720ea43f8 
[root@manager55 ~]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
ob8p30vdkhnn        demo                replicated          4/5                 busybox:latest
# some times later
[root@manager55 ~]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
ob8p30vdkhnn        demo                replicated          5/5                 busybox:latest
```
在manger55上面发现运行机器已经被done了一台，但是一段时间后(不久，相当于重开一台机器的时间)会刷新成5/5 因为scale会检测并重新部署一台新的上去

## 水平扩展后容器的网络问题
之前的docker容器全部都是运行在本地的，于是容器与容器之间是可以通过使用名叫docker0的bridge进行互相通信，但是使用docker swarm之后，容器已经分布式的部署在了多台ip的机器上面，他们的网络是否互相通信呢
```bash
# 查看work233 上的 ip
[work233]:~# docker exec 69d08591e255  ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
266: eth0@if267: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 查看manager 其中一个容器的ip
[root@manager55 ~]# docker exec 0c154e8873c7 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
17: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 测试ping
[root@manager55 ~]# docker exec 0c154e8873c7 ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.186 ms

# 但是ping 容器名就不可达，没有做link
[root@manager55 ~]# docker exec 0c154e8873c7 ping  69d08591e255
ping: bad address '69d08591e255'

```
<font color='red'>Q: 如果解决ping容器名不可达的问题</font>
这是因为在service启动和部署的过程中没有做link，在之后可以部署的过程中指定自己的网桥new-bridge,达到互相通信的效果
具体可以看
https://blog.noback.top/posts/20783356/#linux%E7%9A%84%E7%BD%91%E7%BB%9C%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4namesapce


## 关于服务查看
只能在manager上进行查看，不能在node机上查看，会报错
```bash
[root@work196]# docker service ls
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this comma
nd on a manager node or promote the current node to a manager. 
```