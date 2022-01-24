## Linux下升级内核
最近要搭建一些测试机做测试，版本要求不低于4.19，但是Centos7的内核版本国第，部分功能无法满足，所以需要升级内核

### 备份数据
备份重要数据(例如:/etc ,/var ,/opt文件夹)如果 CentOS 是安装在虚拟机上,那么可以使用快照进行备份.像 VMware 虚拟机可以快照备份.也可以针对重要程序数据进行备份，例如 MySQL、Appache、Nginx、DNS 等等.云主机的话,阿里云和腾讯云都可以创建快照备份数据.

### 升级流程
1.检查版本
```bash
# 检查当前Centos系统版本
cat /etc/redhat-release
# 检查centosntos系统内核版本
uname -r
```
2.更新yum源仓库
```bash
# 清空缓存
yum clean all
# 升级
yum update -y
```
3.启用ELRepo,具体的可以看另一篇yum的文章
```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org 
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
```
4.列出当前可用系统内核相关包
```bash
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
```
5.升级最新的主线内核版本
```bash
yum --enablerepo=elrepo-kernel install kernel-ml
```
6.重启
```bash
reboot
```
7.查看系统上的所有可用内核
```bash
sudo awk -F\' '$1=="menuentry" {print i++ " : " $2}' /etc/grub2.cfg 
```
8.设置默认内核
**方法1**
为了让新安装的内核成为默认启动选项，你需要如下修改 GRUB 配置,打开并编辑 /etc/default/grub 并设置 GRUB_DEFAULT=0.意思是 GRUB 初始化页面的第一个内核将作为默认内核.
```bash
# vi /etc/default/grub

GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=0
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto console=ttyS0 console=tty0 panic=5"
GRUB_DISABLE_RECOVERY="true"
GRUB_TERMINAL="serial console"
GRUB_TERMINAL_OUTPUT="serial console"
GRUB_SERIAL_COMMAND="serial --speed=9600 --unit=0 --word=8 --parity=no --stop=1"
```
**方法2**
```bash
grub2-set-default 0
```
9.重新启用内核配置
```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```
10.列出内核RPM包
```bash
rpm -qa | grep kernel
```
11.删除旧包
```bash
yum remove kernel-xxxx
```
12.使用yum-utils工具，自动删除旧内核
如果安装的内核不多3个,yum-utils工具不会删除任何一个，只有在安装的内核大于3个时，才会自动删除旧内核
```bash
# 安装
yum install yum-utils
# 删除旧版本
package-cleanup --oldkernels
```

## 升级最新内核版本脚本
```bash
#!/bin/bash
# 输出当前版本
echo `uname -r`
# 更新仓库
yum -y update
# 启用ELRpo仓库
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
# 安装最新ml版本kernel
yum --enablerepo=elrepo-kernel install kernel-ml -y
# 安装lt稳定版本kernel
# yum --enablerepo=elrepo-kernel install kernel-lt -y
# 设置grub2
sudo grub2-set-default 0
# 配置并重启
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo reboot

```

## 使用ansible批量升级最新内核版本
```bash
ansible ho -m script -a 'xxx.sh'
```

## 报错内容
yum安装软件报错：curl#6 - "Could not resolve host: mirrorlist.centos.org; Temporary failure in name resolut
**原因**
DNS错误，具体解决查看另一篇yum的文章


## 升级指定内核版本


