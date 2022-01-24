---
title: linux初始化
categories:
  - shell
tags:
  - shell
abbrlink: b4b87579
date: 2020-01-15 10:30:41
---


learn linux 
<!--more-->


## 初始化脚本
```bash
#!/bin/sh

# ---> 脚本初始化

export LC_ALL=en_US.utf8 
# 全局设置为en_US.utf8,为了防止出现乱码
echo -e 'nameserver 192.168.147.20\nnameserver 192.168.21.20\nnameserver 114.114.114.114' > /etc/resolv.conf
# 写入DNS到/etc/resolv.conf 保证能ping通外网

# ---> 设置变量

KERNEL="kernel-ml"
# 设置KERNEL变量，一开始为无 ，reload后为无
HOST="DCK-ZJ-SAD-xxx"
# 设置HOST变量
ZONE="Asia/Shanghai"
# 设置时区变量


# ---> 直接设置

hostnamectl --static set-hostname $HOST
# 修改主机名  也可以直接修改
# echo "DCK-ZJ-SAD-xxx" > /etc/hostname

timedatectl set-timezone $ZONE
# 设置本地时间
timedatectl set-ntp 0
# 关闭ntp同步
timedatectl set-local-rtc 0
# 设置硬件时钟为UTC 

setenforce 0   
# 暂时取消selinux 高安全模式
sed -r -i  '/^SELINUX=/s^=.*^=disabled^g' /etc/selinux/config
# 永久取消selinux
sed -r -i '/^[^root]/s:/bin/bash:/sbin/nologin:g' /etc/passwd
# -r 支持正则
# -i 直接修改文件
# 匹配所有不是root开头的行 将后面的/bin/bash 修改成/sbin/nologin 收回所有除root用户以外的管理者登陆权限
# /^[^root]/ 取反，不是root的行


sed -r -i '/#Port 22/s^.*^Port 65422^g;/^PasswordAuthentication/s^yes^no^g' /etc/ssh/sshd_config
#[root@localhost ~]# sed -r -n '/#Port 22/s^.*^Port 65422^p;/^PasswordAuthentication/s^yes^no^p' /etc/ssh/sshd_config
#Port 65422
#PasswordAuthentication no
# 将登陆端口22 改成65422
# 将密码验证从yes改成no



# --------> ??????????????
sed -r -i -e '/DefaultLimitCORE/s^.*^DefaultLimitCORE=infinity^g' -e '/DefaultLimitNOFILE/s^.*^DefaultLimitNOFILE=100000^g' -e '/DefaultLimitNPROC/s^.*^DefaultLimitNPROC=100000^g' /etc/systemd/system.conf 
#[root@localhost ~]# sed -r -n  -e '/DefaultLimitCORE/s^.*^DefaultLimitCORE=infinity^p' -e '/DefaultLimitNOFILE/s^.*^DefaultLimitNOFILE=100000^p' -e '/DefaultLimitNPROC/s^.*^DefaultLimitNPROC=100000^p' /etc/systemd/system.conf
#DefaultLimitCORE=infinity
#DefaultLimitNOFILE=100000
#DefaultLimitNPROC=100000
# -e 多点编辑
# 好像也可以写成 
# sed -r -i 'xxxx:g;xxx:g;xx:g‘

# /etc/systemd/system.conf 有啥用
# 为什么要这样设置
# 适当放大系统里的资源限制配额
# 还是不懂？



sed -r -i 's@weekly@daily@g;s@^rotate.*@rotate 7@g;s@^#compress.*@compress@g' /etc/logrotate.conf
#root@localhost ~]# sed -r -n 's@weekly@daily@p;s@^rotate.*@rotate 7@p;s@^#compress.*@compress@p ' /etc/logrotate.conf
# rotate log files daily
#daily
#rotate 7
#compress
#将logrotate日志系统设置为  每天执行  保留7天日志  日志进行压缩 等功能开启


# ----------------> ??????????????????
sed -r -i -e '/Compress=/s@.*@Compress=yes@g; /SystemMaxUse=/s@.*@SystemMaxUse=4G@g; ' -e '/SystemMaxFileSize=/s@.*@SystemMaxFileSize=256M@g; /MaxRetentionSec=/s@.*@MaxRetentionSec=2week@g' /etc/systemd/journald.conf
#[root@localhost ~]# sed -r -n -e '/Compress=/s@.*@Compress=yes@p; /SystemMaxUse=/s@.*@SystemMaxUse=4G@p; ' -e '/SystemMaxFileSize=/s@.*@SystemMaxFileSize=256M@p; /MaxRetentionSec=/s@.*@MaxRetentionSec=2week@p' /etc/systemd/journald.conf
#Compress=yes
#SystemMaxUse=4G
#SystemMaxFileSize=256M
#MaxRetentionSec=2week

# /etc/systemd/journald.conf 有啥用



localectl set-locale LANG=en_US.utf8
# 声明语言环境，zh_CN.utf8 中文请款下可能会出现各种乱码
cat > /etc/locale.conf << EOF
LANG=en_US.utf8
LC_CTYPE=en_US.utf8
EOF



cat > /etc/security/limits.d/20-nproc.conf  <<EOF
*          soft    nproc    10240
root       soft    proc     unlimited
EOF
# 限制最大进程数和最大文件打开数
# 这个文件是干嘛的。。

cat > /etc/cron.d/upyun <<EOF
CRON_TZ=$ZONE
0 * * * * root (/usr/sbin/ntpdate -o3 192.168.147.20 211.115.194.21 133.100.11.8 142.3.100.15)
EOF
# 开机启动时加入时间同步，保证系统时间正确
# -o 指定version

yum -u update
yum install -y tree ntpdate telnet bc nc net-tools wget lsof rsync bash-completion iptables-services firewalld sysstat bind-utils python-setuptools yum-utils epel-release smartmontools

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
rpm -Uvh http://download-ib01.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum makecache fast
yum --enablerepo=elrepo-kernel -y install docker-ce
yum clean all
# 各种安装



sed -r -i 's/biosdevname=0//g;s/net.ifnames=0//g' /etc/sysconfig/grub                                                                       
sed -r -i 's/biosdevname=0//g;s/net.ifnames=0//g' /etc/default/grub
$(ip link|grep -iq "eth[0-9]\{1,3\}:.*up") && sed -r -i '/GRUB_CMDLINE_LINUX/s^(.*)="(.*)"$^\1="\2 biosdevname=0 net.ifnames=0"^g' /etc/sysconfig/grub
$(ip link|grep -iq "eth[0-9]\{1,3\}:.*up") && sed -r -i '/GRUB_CMDLINE_LINUX/s^(.*)="(.*)"$^\1="\2 biosdevname=0 net.ifnames=0"^g' /etc/default/grub
ntpdate -o3 192.168.147.20 211.115.194.21 133.100.11.8 142.3.100.15

sed -r -i '$a /usr/sbin/ntpdate -u -o3 192.168.147.20 ntp.aliyun.com 211.115.194.21' /etc/rc.d/rc.local    



ssh-keygen -t rsa -b 4096 -P "" -f ~/.ssh/id_rsa
# 生成key
curl -X GET -u shaohy:Geminis987 -o ~/.ssh/authorized_keys http://devops.upyun.com:88/authorized_keys
# 下载远端key
chmod 0400 ~/.ssh/*
# 修改权限



for file in set_irq.sh set_net_smp_affinity.sh set_rps.sh;do
        curl -X GET -u shaohy:Geminis987 -o /usr/local/sbin/$file http://devops.upyun.com:88/$file
        chmod +x /usr/local/sbin/$file
        sed -r -i "/$file/d" /etc/rc.d/rc.local
        echo "/usr/local/sbin/$file" >> /etc/rc.d/rc.local
done    
chmod +x /etc/rc.d/rc.local

# set.irq.sh set_net_smp_affinity.sh set_rps.sh 这三个在那里？

if [ ! -z $KERNEL ];then
    yum --enablerepo=elrepo-kernel -y install $KERNEL
    Version=`yum info $KERNEL|awk -F: '/Version/{print $2}'`
    Menu=`sed -r -n "s/^menuentry '(.*)' --class.*/\1/p" /boot/grub2/grub.cfg|grep $Version`                                         
    grub2-set-default "$Menu"
    grub2-mkconfig -o /boot/grub2/grub.cfg
fi

#systemctl unmask NetworkManager
#systemctl enable NetworkManager
systemctl daemon-reload
systemctl disable firewalld postfix irqbalance tuned rpcbind.target
systemctl enable auditd 
```


## 安装必要的库
- tree 
目录树
- ntpdate
ntp时间同步工具 
- telnet 
- bc 
科学计算器
- nc 
网络工具
- net-tools
网络工具
- wget 
- lsof 
- rsync 
- bash-completion 
- iptables-services 
- firewalld 
- sysstat 
- bind-utils 
- python-setuptools 
- yum-utils 
- epel-release s
- martmontools

## linux区域设置 locale
locale在linux中用来表示区域之间的差别，它根据文化传统等各个方面分成12个大类，在Linux系统里面，是需要了解的。区域就是世界各地不同区域的意思，LC开头的一些“全局变量”用来进行区域设置，效果就是不同的区域设置，某些程序运行的输出格式不太一样。
- 语言符号及其分类(LC_CTYPE) 
- 数字(LC_NUMERIC) 
- 比较和排序习惯(LC_COLLATE) 
- 时间显示格式(LC_TIME) 
- 货币单位(LC_MONETARY) 
- 信息主要是提示信息,错误信息,状态信息,标题,标签,按钮和菜单等(LC_MESSAGES) 
- 姓名书写方式(LC_NAME) 
- 地址书写方式(LC_ADDRESS) 
- 电话号码书写方式(LC_TELEPHONE) 
- 度量衡表达方式 (LC_MEASUREMENT) 
- 默认纸张尺寸大小(LC_PAPER) 
- 对locale自身包含信息的概述(LC_IDENTIFICATION)

举例
```bash
[root@localhost ~]$ LC_TIME=en_US.UTF-8 date
Fri Feb 22 15:34:10 CST 2019
[root@localhost ~]$ LC_TIME=fi_FI.UTF-8 date
pe 22.2.2019 15.34.20 +0800
[root@localhost ~]$ LC_TIME=zh_CN.UTF-8 date
2019年 02月 22日 星期五 15:34:28 CST

# 使用locale可以查看其中的设置
[root@ops ~]# locale
LANG=zh_CN.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=en_US.UTF-8 <---- 宏定义

# locale.conf记录了缺省值
vi /etc/locale.conf

```
缺省值:LANG是一个缺省值，所有没有显式设置值的LC_*变量都会取LANG的值。

LC_ALL并不是一个环境变量，而是一个glibc中定义的一个宏，LC_ALL=C这样的语法实际上是调用了setlocale把所有的LC_*的变量设置了一遍，所以在终端中直接echo $LC_TIME等可以输出对应变量的值，但是echo $LC_ALL，不好意思，啥都没有，因为它压根就不是一个变量。
定义LC_ALL 既可以一键定义所有的内容,比如初始化
```bash
export LC_ALL=C
```
，

## linux时钟设置 timedatectl

时钟概念
- UTC
整个地球分为二十四时区，每个时区都有自己的本地时间。在国际无线电通信场合，为了统一起见，使用一个统一的时间，称为通用协调时(UTC,Universal Time Coordinated)。
- GMT
格林威治标准时间 (Greenwich Mean Time)指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。(UTC与GMT时间基本相同）
- CST
中国标准时间 (China Standard Time)【GMT + 8 = UTC + 8 = CST】

timedatectl命令可以查询和更改系统时钟和设置，你可以使用此命令来设置或更改当前的日期，时间和时区，或实现与远程NTP服务器的自动系统时钟同步。

```bash
# 查看系统当前时间和日期
root:~# timedatectl status
      Local time: Wed 2020-01-15 19:42:13 CST
  Universal time: Wed 2020-01-15 11:42:13 UTC
        RTC time: Wed 2020-01-15 11:41:51
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: n/a
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a
# 查看Time
root:~# timedatectl | grep Time --color
       Time zone: Asia/Shanghai (CST, +0800)
# 查看所有可用时区
timedatectl list-timezones
# 设置时区
timedatectl set-timezone "Asia/Shanghai"
# 协调世界时
timedatectl set-timezone UTC
# 设置时间
timedatectl set-time 15:38:30
# 设置硬件时钟为本地时间
timedatectl set-local-rtc 1
# 设置硬件时钟为世界时(UTC)
timedatectl set-local-rtc 0
# 查看硬件时钟
timedatectl | grep local --color
```


<font color='red'>将Linux时钟同步到远程NTP服务器</font>
NTP即Network Time Protocol（网络时间协议)

```bash
# 自己当NTP服务器
timedatectl set-ntp true
# 禁用
timedatectl set-ntp false 
```

## Docker容器里时区的同步

linux中`/usr/share/zoneinfo/`下面存放着各个时区 `/etc/localtime` 是本地时间

```bash
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
```




## linux 网络时间协议(Network Time Protocol)  NTP
NTP服务器用来同步时间 ，搭建一台本地的NTP服务器

``` bash
# 安装
yum install -y ntp
# 启动ntp服务
systemctl start ntpd
# 设置开机启动
systemctl enable ntpd
# 查看同步情况
/usr/sbin/ntpq -p 
```

### ntp配置
```bash
# 查看安装ntp工具会产生哪些文件 
root:~# rpm -ql ntp
/etc/dhcp/dhclient.d    
/etc/dhcp/dhclient.d/ntp.sh
/etc/ntp.conf   # ntp配置文件
/etc/ntp/crypto
...

# 配置文件内容
driftfile /var/lib/ntp/drift  #系统时间与BIOS时间的偏差记录
restrict default nomodify notrap nopeer noquery    # 控制相关权限
# default 指带所有ip
# ignore  ：关闭所有的 NTP 联机服务
# nomodify：客户端不能更改服务端的时间参数，但是客户端可以通过服务端进行网络校时。
# notrust ：客户端除非通过认证，否则该客户端来源将被视为不信任子网
# noquery ：不提供客户端的时间查询：用户端不能使用ntpq，ntpc等命令来查询ntp服务器
# notrap ：不提供trap远端登陆：拒绝为匹配的主机提供模式 6 控制消息陷阱服务。陷阱服务是 ntpdq 控制消息协议的子系统，用于远程事件日志记录程序。
# nopeer ：用于阻止主机尝试与服务器对等，并允许欺诈性服务器控制时钟
# kod ： 访问违规时发送 KoD 包。
# restrict -6 表示IPV6地址的权限设置

restrict 127.0.0.1
restrict ::1  # 确保localhost（这个常用的IP地址用来指Linux服务器本身）有足够权限

server 192.168.20.1 perfer # 设定主机来源，perfer定义最优先
restrict 192.168.20.1 nomodify notrap noquery # 允许上有 时间服务器主动修改本机
#server 0.centos.pool.ntp.org iburst  
#erver 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor

# NTP服务需要使用到UDP端口号123

# 初次同步
ntpdate -u 192.168.20.1 
```

## linux科学计算器 bc
科学计算器
```bash
# 安装
yum install bc -y
# 运行计算机
bc -q 
# 借助管道
echo "1+2" | bc
ret=$(echo "4+9" | bc) && echo $ret
```

## linux网络工具  NetCat
```bash
# 安装
yum install nc -y
# 使用
-l Port # 使用监听端口，等待端口活动
-o<输出文件> # 指定文件名称，把往来传输的数据以16进制字码倾倒成该文件保存
-p<通信端口> # 设置本地主机使用的通信端口。
-r # 指定本地与远端主机的通信端口。
-s<来源地址> #设置本地主机送出数据包的IP地址
-u #使用UDP传输协议
-v #显示指令执行过程
-w # 设置等待连线的时间
-z # 使用0输入/输出模式，只在扫描通信端口时使用
```

### 使用
传文件,注意关闭防火墙或者关闭端口
```bash
# 发送端 10.0.5.197
nc 10.0.5.233 8888  < xx.py
# 接收端 10.0.5.233
nc -l 8888 > yy.py
```
端口扫描
```bash
nc -v -w 1 10.0.5.233 -z 20-30 
```
聊天
```bash
# 主机端
nc 10.0.5.233 8888
# 接收端
nc -l 8888 
```

## /etc/logrotate  滚动日志
Linux系统默认安装logrotate工具，它默认的配置文件在：
/etc/logrotate.conf
/etc/logrotate.d/
logrotate.conf 是主要的配置文件，logrotate.d 是一个目录，该目录里的所有文件都会被主动的读入/etc/logrotate.conf中执行。

### 查看安装会产生哪些文件
```bash
[root@DCK-ZJ-SAD-035 logrotate.d]# rpm -ql logrotate
/etc/cron.daily/logrotate
/etc/logrotate.conf  # main
/etc/logrotate.d  # main
/etc/rwtab.d/logrotate
/usr/sbin/logrotate
/usr/share/doc/logrotate-3.8.6
/usr/share/doc/logrotate-3.8.6/CHANGES
/usr/share/doc/logrotate-3.8.6/COPYING
/usr/share/man/man5/logrotate.conf.5.gz
/usr/share/man/man8/logrotate.8.gz
/var/lib/logrotate
/var/lib/logrotate/logrotate.status 
```


### 定时轮循机制
Logrotate是基于CRON来运行的，其脚本是/etc/cron.daily/logrotate，日志轮转是系统自动完成的。

logrotate这个任务默认放在cron的每日定时任务cron.daily下面 /etc/cron.daily/logrotate
/etc/目录下面还有cron.weekly/, cron.hourly/, cron.monthly/ 的目录都是可以放定时任务的

**日志轮询周期根据时间分配有4种**
daily、weekly 、monthly、yearly
默认daily
```bash
cat /etc/cron.daily/logrotate 
#!/bin/sh
/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```


### logrotate配置

全局配置 /etc/logrotate.conf
对应不同程序的配置 /etc/logrotate.d/

<font color='red'>全局配置</font>

```bash
# 轮询时间,可以是weekly,monthly,yearly
daily
# 日志备份保存时间 7天,超过则删除
rotate 7
create   # 创建文件
dateext  # 日志文件切割后添加日期后缀
compress # 切割后压缩
include /etc/logrotate.d  # 指向文件夹
/var/log/wtmp {
    monthly
    create 0664 root utmp
        minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
} 
```

<font color='red'>部分应用程序配置，以nginx为例子</font>

```bash
# 首先需要指定存放nginx日志文件的地址
/usr/local/nginx/logs/access.log  {
daily
rotate 7
missingok  # 在日志轮循期间，任何错误将被忽略，例如“文件无法找到”之类的错
dateext
compress
# npcompress # 如果你不希望对日志文件进行压缩，设置这个参数即可
delaycompress # 总是与compress选项一起用，delaycompress选项指示logrotate不要将最近的归档压缩，压缩将在下一次轮循周期进行
notifempty # 如果日志为空，轮询不进行
sharedscripts  #表示postrotate脚本在压缩了日志之后只执行一次
postrotate
    [ -e /usr/local/nginx/logs/nginx.pid ] && kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
endscript
}
# create 644 www root  # 以指定的权限创建全新的日志文件，同时logrotate也会重命名原始日志文件。

# 执行程序
logrotate /etc/logrotate.d/nginx
```



### 手动使用
```bash
# 如果等不及cron自动执行日志轮转，想手动强制切割日志，需要加-f参数
/usr/sbin/logrotate -f /etc/logrotate.d/nginx
# 正式执行前最好通过Debug选项来验证一下（-d参数）
/usr/sbin/logrotate -d -f /etc/logrotate.d/nginx
```

### logrotate命令格式
```bash
logrotate [OPTION...] <configfile>
-d, --debug ：debug模式，测试配置文件是否有错误，不真实执行。
-f, --force ：强制转储文件。
-m, --mail=command ：压缩日志后，发送日志到指定邮箱。
-s, --state=statefile ：使用指定的状态文件。
-v, --verbose ：显示转储过程。
```
默认将日志保存在/var/lib/logrotate.status文件中

### logrotate原理
- create
- copytruncate





## 地址
logrotate具体
https://wsgzao.github.io/post/logrotate/
locale具体
https://www.cnblogs.com/wn1m/p/10837609.html