---
title: docker-k8s
categories:
  - docker
tags:
  - docker
abbrlink: 7cebdf57
date: 2020-02-10 03:55:11
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker-k8s](#docker-k8s)
  - [安装k8s及其管理工具](#安装k8s及其管理工具)
  - [k8s单节点环境 minikube](#k8s单节点环境-minikube)
    - [可能会出现的错误](#可能会出现的错误)
  - [使用k8s搭建多节点集群](#使用k8s搭建多节点集群)
  - [构建docker+k8s系统环境脚本](#构建dockerk8s系统环境脚本)
  - [使用kubectl](#使用kubectl)

<!-- /code_chunk_output -->
<!-- more -->

# docker-k8s


## 安装k8s及其管理工具

一些需要准备的环境
- 关闭防火墙
```bash
systemctl stop firewalld
systemctl disable firewalld
```
- 禁用selinux
```bash
setenforce 0
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux 
```
- 关闭swap
```bash
swapoff -a
echo "vm.swappiness = 0">> /etc/sysctl.conf  
```
安装k8s 
```bash


# 国外镜像
# 这里的eof会被屏蔽掉所以我在这里加了注释。让他能够完整的显示
# cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# 国内镜像
# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 测试安装地址是否可以使用
curl https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64

# 安装
yum makecache fast
yum install -y kubelet kubeadm kubectl
```

## k8s单节点环境 minikube
1. 检查Linux是否支持虚拟化
```bash
grep -E --color 'vmx|svm' /proc/cpuinfo

# 有两种情况
# 如果不支持虚拟化则显示为空
# 如果支持虚拟化则会显示具体内容，如下
[root@work196 ~]# grep -E --color 'vmx|svm' /proc/cpuinfo
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmo
n pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl  ------> here   ----->  vmx <------- smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc
_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb invpcid_single ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveo
pt dtherm ida arat pln pts md_clear spec_ctrl intel_stibp flush_l1d

```
2. 在linux上直接安装minikube
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
```
3. 启动minikube创建一个k8s单节点集群
```bash
minikube start # 这里可能会出现一个问题， 没有指定虚拟机，
[root@ops ~]# minikube start
😄  minikube v1.7.3 on Centos 7.2.1511
💣  Unable to determine a default driver to use. Try specifying --vm-driver, or see https://minikube.sigs.k8s.io/docs/st

# 解决方法:
minikube start --vm-driver=none
[root@ops ~]# minikube start --vm-driver=none
😄  minikube v1.7.3 on Centos 7.2.1511
   Using the none driver based on user configuration
🤹  Running on localhost (CPUs=2, Memory=1989MB, Disk=20462MB) ...
ℹ️    OS release is CentOS Linux 7 (Core)
🐳  Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
💾  Downloading kubectl v1.17.3
💾  Downloading kubeadm v1.17.3
💾  Downloading kubelet v1.17.3
🚀  Launching Kubernetes ...
...
⚠️  The 'none' driver provides limited isolation and may reduce system security and reliability.
⚠️  For more information, see:
👉  https://minikube.sigs.k8s.io/docs/reference/drivers/none/
```

### 可能会出现的错误
```bash
[root@ops ~]# minikube start --vm-driver=none
😄  minikube v1.7.3 on Centos 7.2.1511
✨  Using the none driver based on user configuration
⌛  Reconfiguring existing host ...
🔄  Starting existing none VM for "minikube" ...
ℹ️   OS release is CentOS Linux 7 (Core)
🐳  Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
🚀  Launching Kubernetes ...

💣  Error starting cluster: init failed. output: "-- stdout --\n[init] Using Kubernetes version: v1.17.3\n[preflight] Running pre-flight checks\n\n-- /stdout --\n** stderr ** \nW0225 12:50:34
.116477   31929 validation.go:28] Cannot validate kube-proxy config - no validator is available\nW0225 12:50:34.116548   31929 validation.go:28] Cannot validate kubelet config - no validator
is available\n\t[WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'\n\t[WARNING IsDockerSystemdCheck]: detected \"cgroupfs\" as the Docker cg
roup driver. The recommended driver is \"systemd\". Please follow the guide at https://kubernetes.io/docs/setup/cri/\n\t[WARNING Hostname]: hostname \"ops.novalocal\" could not be reached\n\t
[WARNING Hostname]: hostname \"ops.novalocal\": lookup ops.novalocal on 114.114.114.114:53: no such host\n\t[WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl en
able kubelet.service'\nerror execution phase preflight: [preflight] Some fatal errors occurred:\n\t[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridg
e-nf-call-iptables contents are not set to 1\n[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`\nTo see the stack trace of this e
rror execute with --v=5 or higher\n\n** /stderr **": /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.17.3:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-prefli
ght-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable-
-etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap,SystemVeri
fication": exit status 1
stdout:
[init] Using Kubernetes version: v1.17.3
[preflight] Running pre-flight checks

stderr:
W0225 12:50:34.116477   31929 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0225 12:50:34.116548   31929 validation.go:28] Cannot validate kubelet config - no validator is available
        [WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING Hostname]: hostname "ops.novalocal" could not be reached
        [WARNING Hostname]: hostname "ops.novalocal": lookup ops.novalocal on 114.114.114.114:53: no such host
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher


😿  minikube is exiting due to an error. If the above message is not useful, open an issue:
👉  https://github.com/kubernetes/minikube/issues/new/choose
```
这里报错说是/proc/sys/net/bridge/bridge-nf-call-iptables没有设置1 
解决方法
```bash
echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables
```

<font color='red'>minikube搭建的单节点的过程中会需要一些时间，可能就卡着不动了</font>

安装完成
```bash
[root@ops ~]# minikube start --vm-driver=none
😄  minikube v1.7.3 on Centos 7.2.1511
✨  Using the none driver based on user configuration
⌛  Reconfiguring existing host ...
🏃  Using the running none "minikube" VM ...
ℹ️   OS release is CentOS Linux 7 (Core)
🐳  Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
🚀  Launching Kubernetes ...

🌟  Enabling addons: default-storageclass, storage-provisioner
🤹  Configuring local host environment ...

⚠️  The 'none' driver provides limited isolation and may reduce system security and reliability.
⚠️  For more information, see:
👉  https://minikube.sigs.k8s.io/docs/reference/drivers/none/

⚠️  kubectl and minikube configuration will be stored in /root
⚠️  To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:

    ▪ sudo mv /root/.kube /root/.minikube $HOME
    ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

💡  This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
🏄  Done! kubectl is now configured to use "minikube"
```

<font color='red'>使用minikub ssh 进入容器，如果你是在Linux机器上的还，是无法使用minikub ssh 登陆的，因为你已经在minikue节点上了</font>


## 使用k8s搭建多节点集群

机器分配
10.0.6.55 master
10.0.5.233 node1
10.0.5.196 node2


kubeadm init on master node
```bash
sudo kubeadm init --pod-network-cidr 172.100.0.0/16 --apiserver-advertise-address 192.168.205.120  # 192.168.205.120 is master node ip

# result
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.6.55:6443 --token dbw4qc.ey3dpkozmv8yb15t \
    --discovery-token-ca-cert-hash sha256:b1c47bc371709d90c6c381e6f3c7c9c034a7dcbfc585b18260016d480aab24a8
```
继续运行
```bash
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
检查pod 
```bash
[root@manager55 ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-2d7b4            0/1     Pending   0          7m34s   # 这两个是网络插件后面安装
kube-system   coredns-6955765f44-hmg9g            0/1     Pending   0          7m34s
kube-system   etcd-manager55                      1/1     Running   0          7m49s
kube-system   kube-apiserver-manager55            1/1     Running   0          7m49s
kube-system   kube-controller-manager-manager55   1/1     Running   0          7m49s
kube-system   kube-proxy-v8dxm                    1/1     Running   0          7m34s
kube-system   kube-scheduler-manager55            1/1     Running   0          7m49s
```
安装网络插件
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')" 
```
检查网络插件
```bash
[root@manager55 ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-2d7b4            1/1     Running   0          9m1s
kube-system   coredns-6955765f44-hmg9g            1/1     Running   0          9m1s
kube-system   etcd-manager55                      1/1     Running   0          9m16s
kube-system   kube-apiserver-manager55            1/1     Running   0          9m16s
kube-system   kube-controller-manager-manager55   1/1     Running   0          9m16s
kube-system   kube-proxy-v8dxm                    1/1     Running   0          9m1s
kube-system   kube-scheduler-manager55            1/1     Running   0          9m16s
kube-system   weave-net-bf5jp                     2/2     Running   0          42s 
```

将其他node机器加入到master主机群下面
```bash
kubeadm join 10.0.6.55:6443 --token dbw4qc.ey3dpkozmv8yb15t \
    --discovery-token-ca-cert-hash sha256:b1c47bc371709d90c6c381e6f3c7c9c034a7dcbfc585b18260016d480aab24a8 
```
在master主机上查看node节点加入情况
```bash
# 加入node 前
[root@manager55 ~]# kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
manager55   Ready    master   11m   v1.17.2 
# 加入之后
[root@manager55 ~]# kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
manager55   Ready    master   60m   v1.17.2
work196     Ready    <none>   34m   v1.17.2
work233     Ready    <none>   15m   v1.17.2
```


## 构建docker+k8s系统环境脚本

<font color='red'>centos7版本</font>

```bash
#/bin/sh

# install some tools
sudo yum install -y vim telnet bind-utils wget


# install docker
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh

if [ ! $(getent group docker) ];
then·
    sudo groupadd docker;
else
    echo "docker user group already exists"
fi

sudo gpasswd -a $USER docker
sudo systemctl restart docker

rm -rf get-docker.sh

# open password auth for backup if ssh key doesn't work, bydefault, username=vagrant password=vagrant
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
sudo systemctl restart sshd

sudo bash -c 'cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF'

# 也可以尝试国内的源 http://ljchen.net/2018/10/23/%E5%9F%BA%E4%BA%8E%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E7%AB%99%E5%AE%89%E8%A3%85ku⟫

sudo setenforce 0

# install kubeadm, kubectl, and kubelet.
sudo yum install -y kubelet kubeadm kubectl

sudo bash -c 'cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
EOF'
sudo sysctl --system

sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo swapoff -a

systemctl enable docker.service
systemctl enable kubelet.service 
```



## 使用kubectl
在linux下面使用

```bash
# 查看补足帮助，里面会给出一些各个环境的补足帮助
kubectl  completion  -h
# linux下的补足
source <(kubectl completion bash)
```