---
title: docker-k8s 基础
categories:
  - docker
tags:
  - docker
abbrlink: b83f2a68
date: 2020-02-25 15:52:22
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker-k8s基础](#docker-k8s基础)
  - [k8s中的最小调度单元 pod](#k8s中的最小调度单元-pod)
    - [根据pods yaml文件创建一个nginx容器](#根据pods-yaml文件创建一个nginx容器)
    - [批量创建多个容器 ReplicaSet](#批量创建多个容器-replicaset)
    - [横向扩展与缩放](#横向扩展与缩放)
  - [kubectl的使用方法](#kubectl的使用方法)

<!-- /code_chunk_output -->
<!-- more -->



# docker-k8s基础

## k8s中的最小调度单元 pod
![2020-02-25-15-54-43](http://noback.upyun.com/2020-02-25-15-54-43.png)
在k8s里面，有很多新的技术概念，其中有一个东西被称之为pod。pod是k8s集群里面运行和部署的最小单元，它的设计理念是，一个pod可以承载多个容器，并且共享网络地址和文件系统，内部的容器通过进程间的通信相互访问。
也就是说在k8s中，最小的单位不再是container，而是pod 
一个pod节点中可以有多个container容器


### 根据pods yaml文件创建一个nginx容器
创建一个pod节点，pod.yml文件
```yml
## pod_nginx.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
  ¦ app: nginx
spec:
  containers:
  - name: nginx
  ¦ image: nginx
  ¦ ports:
  ¦ - containerPort: 80
```
创建pod节点
```bash
# 创建节点
[root@ops ~]# kubectl create -f pod_nginx.yml
pod/nginx created
# 查看节点
[root@ops ~]# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2m27s
# 查看详细信息
[root@ops ~]# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE            NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          4m44s   172.17.0.4   ops.novalocal   <none>           <none>
```

<font color='red'>查看容器id</font>

如果你本身自己就是linux机器的话，机器本身就是Minikub节点了，查看docker
```bash
[root@ops ~]# docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
be7d61df08ee        nginx                  "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes                            k8s_nginx_nginx_default_3a2d631d-0de5-40fc-969d-1451a2190389_0
55af5366716a        k8s.gcr.io/pause:3.1   "/pause"                 5 minutes ago       Up 5 minutes                            k8s_POD_nginx_default_3a2d631d-0de5-40fc-969d-1451a2190389_0
## 进入容器,查看网络
docker exec be7d61df08ee -it /bin/bash
```

### 批量创建多个容器 ReplicaSet
```yml
## rs_nginx.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
  labels:
  ¦ tier: frontend
spec:
  replicas: 3
  selector:
  ¦ matchLabels:
  ¦ ¦ tier: frontend
  template:
  ¦ metadata:
  ¦ ¦ name: nginx
  ¦ ¦ labels:
  ¦ ¦ ¦ tier: frontend
  ¦ spec:
  ¦ ¦ containers:
  ¦ ¦ - name: nginx
  ¦ ¦ ¦ image: nginx
  ¦ ¦ ¦ ports:
  ¦ ¦ ¦ - containerPort: 80
```
创建多节点
```bash
kubectl create -f rs_nginx.yml
[root@ops ~]# kubectl get pods
NAME          READY   STATUS              RESTARTS   AGE
nginx-df6pr   1/1     Running             0          10s
nginx-sspd2   0/1     ContainerCreating   0          10s
nginx-vbjs6   0/1     ContainerCreating   0          10s
```

<font color='blue'>使用这个方法创建的内容，会维持容器的数量</font>

```bash
## 删除一个容器
[root@ops ~]# kubectl delete pods nginx-df6pr
pod "nginx-df6pr" deleted
# 再次查看节点中的容器时，会发现他会再生成一个容器
[root@ops ~]# kubectl get pods
NAME          READY   STATUS              RESTARTS   AGE
nginx-pnk4c   0/1     ContainerCreating   0          4s
nginx-sspd2   1/1     Running             0          5m51s
nginx-vbjs6   1/1     Running             0          5m51s
```

### 横向扩展与缩放
将容器减少到2个
```bash
[root@ops ~]# kubectl scale rs nginx --replicas=2
replicaset.apps/nginx scaled
[root@ops ~]# kubectl get rs
NAME    DESIRED   CURRENT   READY   AGE
nginx   2         2         2       21m
[root@ops ~]# kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
nginx-sspd2   1/1     Running   0          21m
nginx-vbjs6   1/1     Running   0          21m
```
将容器增加到5个
```bash
[root@ops ~]# kubectl scale rs nginx --replicas=2
replicaset.apps/nginx scaled
[root@ops ~]# kubectl scale rs nginx --replicas=5
replicaset.apps/nginx scaled
[root@ops ~]# kubectl get pods
NAME          READY   STATUS              RESTARTS   AGE
nginx-bnnbc   0/1     ContainerCreating   0          3s
nginx-jh5kn   0/1     ContainerCreating   0          3s
nginx-skhqq   0/1     ContainerCreating   0          3s
nginx-sspd2   1/1     Running             0          23m
nginx-vbjs6   1/1     Running             0          23m
```

## kubectl的使用方法
kubectl提供了许多便捷的方法，其帮助命令也很完善，这边举例几个常用的
```bash
kubectl get pods -o wide # 查看pods节点上有哪些容器
kubectl exec -it CONTAINER-NAME sh # 进入第一个叫CONTAINER-NAME的容器
kubectl exec -it -c Container name sh # 指定容器名字进入
kubectl describe pods nginx # 查看pods节点详情
kubectl get pods # 查看有哪些节点
kubectl create -f xx.yml # 根据yml文件创建pods
kubectl delete -f xx.yml # 删除由xx.yml创建的pods
```

