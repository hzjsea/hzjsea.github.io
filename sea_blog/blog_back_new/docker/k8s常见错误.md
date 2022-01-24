---
title: k8s常见错误
categories:
  - docker
tags:
  - docker
abbrlink: 6e725c12
date: 2020-02-10 14:04:28
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [k8s常见错误](#k8s常见错误)
  - [k8s错误](#k8s错误)
  - [初始化环境过程中node子节点出现已加入master集群的情况](#初始化环境过程中node子节点出现已加入master集群的情况)
  - [k8s pods拉取镜像时，出现证书错误的问题](#k8s-pods拉取镜像时出现证书错误的问题)
  - [docker错误](#docker错误)
    - [docker安装过程中发生错误](#docker安装过程中发生错误)

<!-- /code_chunk_output -->
<!-- more -->


# k8s常见错误

由于k8s的使用过程中，会出现很多的问题，预计篇幅会很大，在这里做一个记录


## k8s错误
## 初始化环境过程中node子节点出现已加入master集群的情况
```bash 
W0210 13:47:24.391420    9455 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag i
s not set.
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the
 guide at https://kubernetes.io/docs/setup/cri/
        [WARNING Hostname]: hostname "work233" could not be reached
        [WARNING Hostname]: hostname "work233": lookup work233 on 114.114.114.114:53: no such host
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR DirAvailable--etc-kubernetes-manifests]: /etc/kubernetes/manifests is not empty
        [ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher 
```

<font color='red'>解决方法</font>
重置k8s
```bash
kubeadm reset
```
另外 WARNING不大影响，可以选择不修改，ERROR的错误会直接导致无法初始化环境


## k8s pods拉取镜像时，出现证书错误的问题
```bash
FailedSynError syncing pod, skipping: failed to "StartContainer" for "POD" with ErrImagePull: "image pull failed for registry.access.redhat.com/rhel7/pod-infrastructure:latest, this may be because there are no credentials on this request.  details: (open /etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt: no such file or directory)" 13m 11s 56 {kubelet 127.0.0.1} Warning FailedSync Error syncing pod, skipping: failed to "StartContainer" for "POD" with ImagePullBackOff: "Back-off pulling image \"registry.access.redhat.com/rhel7/pod-infrastructure:latest\""

Error syncing pod, skipping: failed to "StartContainer" for "POD" with ErrImagePull: "image pull failed for registry.access.redhat.com/rhel7/pod-infrastructure:latest, this may be because there are no credentials on this request.  details: (open /etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt: no such file or directory)"
```

大致意思就是：未能通过ErrImagePull为“POD”启动“StartContainer”：“对于registry.access.redhat.com/rhel7/pod-infrastructure:latest，图像拉出失败，这可能是因为此请求上没有证书 

检查发现：/etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt这个目录中是一个软连接

![](https://ask.qcloudimg.com/http-save/yehe-1031585/3jzqu3w9wh.png?imageView2/2/w/1620)

但是在系统中没有这个文件的存在
![](https://ask.qcloudimg.com/http-save/yehe-1031585/uq2eoxcxk7.png?imageView2/2/w/1620)

安装证书
```bash
yum remove *rhsm* -y 
yum install *rhsm* -y 
```
使用包安装
```bash
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm
rpm2cpio python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm | cpio -iv --to-stdout ./etc/rhsm/ca/redhat-uep.pem | tee /etc/rhsm/ca/redhat-uep.pemwget http://mirror.centos.org/centos/7/os/x86_64/Packages/python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm
rpm2cpio python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm | cpio -iv --to-stdout ./etc/rhsm/ca/redhat-uep.pem | tee /etc/rhsm/ca/redhat-uep.pem
```
## docker错误
### docker安装过程中发生错误
```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
File "/bin/yum-config-manager", line 133
except yum.Errors.RepoError, e:
```
<font color='red'>解决办法,python环境问题</font>
```bash
sudo vim /bin/yum-config-manager
#!/usr/bin/python -tt   改成 #!/usr/bin/python2 -tt 
```
