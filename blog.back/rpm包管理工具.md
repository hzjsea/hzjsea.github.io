---
title: rpm 与 yum
categories:
  - linux
tags:
  - linux
abbrlink: fa23aabb
date: 2020-02-24 21:59:43
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [rpm 与 yum](#rpm-与-yum)
  - [rpm](#rpm)
    - [RPM工作原理](#rpm工作原理)
    - [rpm 安装路径](#rpm-安装路径)
    - [RPM的指令](#rpm的指令)
    - [rpm验证与数码签章(Verify/signature)](#rpm验证与数码签章verifysignature)
      - [rpm验证](#rpm验证)
  - [yum](#yum)
    - [常用命令](#常用命令)
    - [yum的配置文件](#yum的配置文件)
  - [安装mysql实例](#安装mysql实例)

<!-- /code_chunk_output -->
<!-- more -->

# rpm 与 yum

## rpm
RPM是RedHat Package Manager（RedHat软件包管理工具）类似Windows里面的“添加/删除程序”。rpm 执行安装包二进制包（Binary）以及源代码包（Source）两种。二进制包可以直接安装在计算机中，而源代码包将会由RPM自动编译、安装。源代码包经常以src.rpm作为后缀名。RPM 现在是 Linux Standard Base (LSB) 中采用的包管理系统。

<font color='red'>rpm更多的用来做查询 检验，在小部分上做安装，大部分上使用yum安装</font>

### RPM工作原理
他最大的特点就是将你要安装的软件先编译过， 并且打包成为 RPM 机制的包装文件，通过包装好的软件里头默认的数据库记录， 记录这个软件要安装的时候必须具备的相依属性软件，当安装在你的 Linux 主机时， RPM 会先依照软件里头的数据查询 Linux 主机的相依属性软件是否满足， 若满足则予以安装，若不满足则不予安装。
优点:
- 由于已经编译完成并且打包完毕，所以软件传输与安装上很方便 （不需要再重新编译）；
- 由于软件的信息都已经记录在 Linux 主机的数据库上，很方便查询、升级与安装

### rpm 安装路径
一般来说，RPM 类型的文件在安装的时候，会先去读取文件内记载的设置参数内容，然后将该数据用来比对 Linux 系统的环境，以找出是否有属性相依的软件尚未安装的问题。例如 Openssh 这个连线软件需要通过 Openssl 这个加密软件的帮忙，所以得先安装 openssl 才能装 openssh 的意思。那你的环境如果没有 openssl ， 你就无法安装 openssh。
若环境检查合格了，那么 RPM 文件就开始被安装到你的 Linux 系统上。安装完毕后，该软件相关的信息就会被写入 /var/lib/rpm/ 目录下的数据库文件中了。 上面这个目录内的数据很重要喔！因为未来如果我们有任何软件升级的需求，版本之间的比较就是来自于这个数据库， 而如果你想要查询系统已经安装的软件，也是从这里查询的！同时，目前的 RPM 也提供数码签章信息， 这些数码签章也是在这个目录内记录的呢！所以说，这个目录得要注意不要被删除了
```bash
ll /var/lib/rpm/

[root@ops rpm]# ll
total 35212
-rw-r--r--. 1 root root  1638400 Feb 24 15:46 Basenames
-rw-r--r--. 1 root root     8192 Feb 24 15:45 Conflictname
-rw-r--r--  1 root root   286720 Feb 24 20:27 __db.001
-rw-r--r--  1 root root    90112 Feb 24 20:27 __db.002
-rw-r--r--  1 root root  1318912 Feb 24 20:27 __db.003
-rw-r--r--. 1 root root   598016 Feb 24 15:46 Dirnames
-rw-r--r--. 1 root root     8192 Feb 24 15:46 Group
-rw-r--r--. 1 root root    12288 Feb 24 15:46 Installtid
-rw-r--r--. 1 root root    28672 Feb 24 15:46 Name
-rw-r--r--. 1 root root    16384 Feb 24 15:46 Obsoletename
-rw-r--r--. 1 root root 29913088 Feb 24 15:46 Packages
-rw-r--r--. 1 root root  2019328 Feb 24 15:46 Providename
-rw-r--r--. 1 root root   155648 Feb 24 15:46 Requirename
-rw-r--r--. 1 root root    40960 Feb 24 15:46 Sha1header
-rw-r--r--. 1 root root    24576 Feb 24 15:46 Sigmd5
-rw-r--r--. 1 root root     8192 Feb 10 16:46 Triggername
```

### RPM的指令
RPM的指令太多，使用时又经常是组合的形式存在。
```bash
rpm -ivh xxx.rpm #安装并 显示进度  --install--verbose--hash
rpm -ivh xxx.rpm yyy.rpm # 同时安装多个
rpm -Uvh xxx.rpm #升级软件包并 显示进度  --Update--verbose-hash
rpm -qpl xxx.rpm #列出RPM软件包内的文件信息
rpm -qpi xxx.rpm #列出RPM软件包的描述信息
rpm -qf xxx.rpm #查找指定文件属于哪个RPM软件包
rpm -Va xxx.rpm #检验所有的RPM软件包，查找丢失的文件
rpm -e xxx.rpm #删除包
rpm -e --nodeps xxx.rpm #忽略依赖的检查来删除,但最好不要这么做，因为忽略依赖，会变得很杂乱
rpm --import RPM-GPG-KEY(签名文件) #导入签名
```

### rpm验证与数码签章(Verify/signature)

#### rpm验证
使用 /var/lib/rpm 下面的数据库内容来比对目前 Linux 系统的环境下的所有软件文件,如果存在改动则会列出来

```bash
rpm -Va # 检验所有已安装的软件
rpm -V + 已安装的软件名称 # 定向检验某个软件
rpm -Vp + 某个 RPM 文件的文件名  # 列出该软件内可能被更动过的文件
rpm -Vf # 列出某个文件是否被更动过
``` 

<font color='red'>如果和确认文件更新的情况</font>

举例
```bash
[root@ops rpm]# rpm -Vf /etc/logrotate.conf
S.5....T.  c /etc/logrotate.conf
```

c就是配置文件的意思(configuration)
```bash
SM5DLUGTP c filename
S ：（file Size differs） 文件的容量大小是否被改变
M ：（Mode differs） 文件的类型或文件的属性 （rwx） 是否被改变？如是否可执行等参数已被改变
5 ：（MD5 sum differs） MD5 这一种指纹码的内容已经不同
D ：（Device major/minor number mis-match） 设备的主/次代码已经改变
L ：（readLink（2） path mis-match） Link 路径已被改变
U ：（User ownership differs） 文件的所属人已被改变
G ：（Group ownership differs） 文件的所属群组已被改变
T ：（mTime differs） 文件的创建时间已被改变
P ：（caPabilities differ） 功能已经被改变
```
另外c还有其他的几种文件类型
```bash
c ：配置文件 （config file）
d ：文件数据文件 （documentation）
g ：鬼文件～通常是该文件不被某个软件所包含，较少发生！（ghost file）
l ：授权文件 （license file）
r ：读我文件 （read me）
```

## yum
既然看了rpm,那么与他紧密相关的yum也值得一看, yum 是通过分析 RPM 的标头数据后， 根据各软件的相关性制作出属性相依时的解决方案，然后可以自动处理软件的相依属性问题，以解决软件安装或移除与升级的问题。

### 常用命令
```bash
yum search nginx # 搜寻某个软件名称或者是描述 （description） 的重要关键字；
yum list nginx  # 列出目前 yum 所管理的所有的软件名称与版本，有点类似 rpm -qa；
yum info jq # 查看包信息
yum list update # 列出目前服务器上可供本机进行升级的软件有哪些？
yum install jq --installroot=/some/path #将该软件安装在 /some/path 而不使用默认路径
yum remove jq # 删除
yum clean all # 经所有软件库数据删掉，清除缓存
yum repolist all # 显示所有仓库
yum-config-manager --enable repository... # 打开仓库
yum-config-manager --disable repository... # 关闭仓库
```

### yum的配置文件
在你的linux生成后，系统会默认给你几个 CentOS 的映射站台，但是这些CentOS 的映射站台可能会选错，或者说速度会很慢，那么我们就要修改我们的yum配置文件
```bash
vim /etc/yum.repos.d/CentOS-Base.repo
[base]
# 代表软件库的名字！中括号一定要存在，里面的名称则可以随意取。但是不能有两个相同的软件库名称， 否则 yum 会不晓得该到哪里去找软件库相关软件清单文件。
name=CentOS-$releasever - Base
# 只是说明一下这个软件库的意义而已，重要性不高！
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
# 列出这个软件库可以使用的映射站台，如果不想使用，可以注解到这行；
baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
# 这个最重要，因为后面接的就是软件库的实际网址！ mirrorlist 是由 yum 程序自行去捉映射站台， baseurl 则是指定固定的一个软件库网址！我们刚刚找到的网址放到这里来啦！
gpgcheck=1
# 指定是否需要查阅 RPM 文件内的数码签章
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
# 就是数码签章的公钥档所在位置！使用默认值即可
```
更新成阿里云的repo
```bash
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```


## 安装mysql实例
1. 添加yum源，为了下载的速度，我们添加一个阿里源
```bash
wget https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-connectors-community-el7/mysql-community-release-el7-5.noarch.rpm
```
2. 安装镜像源
```bash
rpm -ivh mysql-community-release-el7-5.noarch.rpm
```
安装后/etc/yum.repos.d/目录下多了两个文件
```bash
[root@work196 yum.repos.d]# ll  grep 'mysql' --color
-rw-r--r--. 1 root root 1060 Jan 29  2014 mysql-community-source.repo
-rw-r--r--. 1 root root 1209 Jan 29  2014 mysql-community.repo
```
3. 清理yum缓存，重新建立缓存
```bash
yum clean all
yum makecache
```
4. 关闭8.0镜像,开启5.7镜像
```bash
yum-config-manager --disable mysql80-community
yum-config-manager --enable mysql57-community
```
5. 下载安装
```bash
yum install mysql-community-server
```
6. 启动
```bash
systemctl start mysqld
```