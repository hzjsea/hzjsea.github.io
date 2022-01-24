Linux作为一个流行的操作系统，对于不同的操作系统，其下载工具或命令也不同，在Centos中 yum作为其常用的操作系统。

## 什么是yum
yum是Yellowdog update Modified的简称。yum的宗旨是自动化的升级、安装/移除rpm安装包（也就是说yum这个命令它的操作对象是RPM包），收集rmp的相关信息，检查依赖性，并提示用户解决。yum的关键之处是要有可靠的repository，顾名思义这就是软件的仓库，它可以是http或者ftp站点，也可以是本地的软件池，但是必须包含rpm的header，rmp的header包括了rmp的各种信息，包括描述、功能、提供的文件、依赖性等，正是收集了这些信息，才能自动化的完成余下的任务。yum本身就是运行在linux上的自动管理安装包的系统。yum 的理念是使用一个中心仓库(repository)管理一部分甚至一个distribution 的应用程序相互关系，根据计算出来的软件依赖关系进行相关的升级、安装、删除等等操作，减少了Linux 用户一直头痛的dependencies 的问题。这一点上，yum 和apt 相同。apt 原为debian 的deb 类型软件管理所使用，但是现在也能用到RedHat 门下的rpm 了。
　　注：linux下的RPM的全称是“RedHat Package Manager”，最早是Red Hat公司开发的，后来在CentOS、Fedora、SUSE都用它。而rpm包则是软件编译完成后按照RPM机制打包起来的一个文件，可以用rpm命令安装的一个软件安装包，它省去了Linux软件安装中编译的步骤，安装成功后软件就可以用了。RPM包的特点是：1.实现已经编译好；2.安装方便；3.安装过程中要求环境一致；4.反安装时要从最上层开始。RPM包的名称规则示例：ttpd-manual- 2.0.40-21.i386.rpm，ttp-manual是软件包的名称，2是主版本号；0是次版本号；40是次版本号；21是编译的次数；i386是适合的平台；.rpm说明这是一个RPM包
rpm作为软件包的编译工具，但对于一个大型的软件来说他所需要的依赖会很多，使用者需要根据依赖列表去一个个手动安装这些依赖，直到最后软件安装完成，这样对于使用者来说无疑是一种麻烦，于是yum这个下载工具就出现了，他主要目的是管理这些rpm包，这些rpm包会放在一些资源库(repository)中，特点1,可以同时配置多个repository中； 特点2,简洁的配置文件(一般配置文件在/etc/yum.conf)中 ； 特点3 自动解决增加或者删除rpm包时遇到的依赖问题 特点4 保持和rpm的数据库一致

## yum的配置文件
yum的配置文件一般存放在yum.conf中，使用命令cat /etc/yum.conf查看文件内容
```bash
[main]
cachedir=/var/cache/yum/$basearch/$releasever   # 此项为yum下载的RPM包的缓存目录，yum在此存储下载的rpm包和数据库
keepcache=0  # 缓存是否保存，1表示安装后保留安装包  0 表示删除
debuglevel=2 # 出错日志 级别为0-10 默认为2(只保留安装和删除记录)
logfile=/var/log/yum.log #存放系统更新软件的记录
exactarch=1 # 有两个选项1和0，代表是否只升级和你安装软件包cpu体系一致的包，如果设为1，则如你安装了一个i386的rpm，则yum不会用1686的包来升级。
obsoletes=1 # 是否允许更新陈旧的RPM包
gpgcheck=1  # 一种密钥方式签名
plugins=1 # 是否允许使用插件，默认是0不允许
installonly_limit=5 
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release


#  This is the default, if you make this bigger yum won't see if the metadata
# is newer on the remote and so you'll "gain" the bandwidth of not having to
# download the new metadata and "pay" for it by yum not having correct
# information.
#  It is esp. important, to have correct metadata, for distributions like
# Fedora which don't keep old packages around. If you don't like this checking
# interupting your command line usage, it's much better to have something
# manually check the metadata once an hour (yum-updatesd will do this).
# metadata_expire=90m

# PUT YOUR REPOS HERE OR IN separate files named file.repo
# in /etc/yum.repos.d

```
yum.conf一般分为main和repository两部分，但是默认情况下只有main部分。每一个yum.conf都只能有一个main部分。repository 部分定义了每个源/服务器的具体配置，可以有一到多个。常位于/etc/yum.repo.d 目录下的各文件中。
使用  ls /etc/yum.repo.d/ 查看文件夹下来存在哪些仓库文件


## 工作原理
yum的工作模式是C/S架构
Server端 ： 依赖关系库、原文件、校验码文件
Client端 :  yum客户端程序、配置文件
执行yum命令时，会首先从”/etc/yum.repo.d”目录下的众多repo文件中取得软件仓库的地址并下载“元数据”，“元数据”包含注册于该软件仓库内所有软件包的包名及其所需的依赖环境等信息，yum得到这些信息后会和本地以后环境做对比，进而列出确认需要安装哪些包，并在用户确认后开始安装。服务器端：在服务器上面存放了所有的rpm软件包，然后以相关的功能去分析每个rpm文件的依赖性关系，将这些数据记录成文件存放在服务器的某特定目录内。客户端：如果需要安装某个软件时，先下载服务器上面记录的依赖性关系文件(可通过WWW或FTP方式)，通过对服务器端下载的纪录数据进行分析，然后取得所有相关的软件，一次全部下载下来进行安装。

## repo文件
前面说到过，现在我们要下载一个软件A，但这个软件A有一系列的依赖列表，我们不想要手动的去一个一个的安装这些依赖，这时候我们有了yum这个工具，它可以将这些依赖列表一个一个的放进一个文件中整合起来，我们把它叫做repo文件，默认情况下，安装完成Centos中有这三个.repo文件 
```bash
CentOS-Debuginfo.repo  debug包尤其和内核相关的更新和软件安装
CentOS-Media.repo 这个是使用光盘挂载后调用的文件（我机器上没有）
CentOS-Vault.repo  这个是最近新版本的加入的老版本的yum源配置（没有。。。）
epel.repo 
```
epel.repo：EPEL（Extra Packages for Enterprise Linux）是由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS等提供高质量软件包的项目。装上了 EPEL，就像在 Fedora 上一样，可以通过 yum install 软件包名，即可安装很多以前需要编译安装的软件、常用的软件或一些比较流行的软件，比如现在流行的nginx、htop、ncdu、vnstat等等，都可以使用EPEL很方便的安装更新。前可以直接通过执行命令： yum install epel-release 直接进行安装，如果不能安装，参考：https://www.cnblogs.com/imweihao/p/7357484.html



## repo配置内容
```text
# CentOS-Base.repo
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```
这里我们那CentOS-Base.repo来做介绍

```bash
#[serverid]是用于区别各个不同的repository，必须有一个独一无二的名称。若重复了，则是后面覆盖前面的
[extras]
#name，是对repo的描述
name = name for server
# baseurl是服务器设置中最重要的部分，只有设置正确，才能从上面获取软件，记住有#号,且必须指向responsitory目录的上一级
\#baseurl=url://server1/path/to/repository/
\#url://server2/path/to/repository/
\#url://server3/path/to/repository/
#其中url支持的协议有http ftp file三种。但是不能以如下形式出现
\#baseurl=url://server1/path/to/repository/
\#baseurl=url://server2/path/to/repository/
\#baseurl=url://server3/path/to/repository/
#enabled={1|0} #是否启用此yum仓库，默认启用
#gpgcheck={1|0} # 是否检查完整性和来源合法性
#gpgkey=url  #密钥文件位置，可能 是对方仓库提供
#enablegroups={1|0} #是否基于组来批量管理程序包
```
1)\*.repo可以将多个[repo id]的配置信息放在一个文件内，也可以切成多个方便管理
2)baseurl可以使用：ftp:// 、http:// 、nfs:// 、file:///  指明URL路径
3)baseurl等号两边不能有空格，其后可以填写多个镜像访问路径，每行一个，不能顶行写多个访问路径间联系是镜像相同，目的是为了做备用访问
4)更多选项使用man  5  yum.conf查看，基本配置只需前三行就可以        
5)发行版光盘镜像安装可能会自动配置网络镜像URL地址


```bash
OPTIONS:
 --nogpgcheck : 禁止gpg check
	-y : 默认yes
	-q : 静默模式，不输出显示信息
	--disablerepo=xxx 临时禁用此处repo
	--enablerepo=xxx 临时启用此处repo
	--noplugins: 禁用所有插件
 repolist [all|enabled|disabled] 显示仓库列表(所有|可用|不可用) 
```