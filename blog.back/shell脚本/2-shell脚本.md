# 脚本合集
# linux shell script

## 变量
linux中的变量分为很多种类，其中shell变量包括两种
1. shell变量
局部变量指的是在脚本中创建，运行在脚本中的变量,Bshell无法访问Ashell中定义的局部变量
定义方式:
```
A3="test"
delcare A3="test" 
```
其中delcare可以省略
2. 用户的环境变量
所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
通过export暴露当前shell的变量
```
A1="123"
export A1
export A1="123" 
```
## 判断行数
判断用户名文件的行数是否大于25，如果大于25，提示 row above 25.  一行实现
```bash
if [ `wc -l /etc/passwd |awk '{print $1}'` -gt 25 ]; then echo "row above 25"; fi 
```
## 替换py2.7 
修改yum中的内容为支持python2.7
```bash
sed -i 's:bin/python:bin/python2.7:' /usr/bin/yum
cat /usr/bin/yum
```
##  逐行处理文本文件
利用read读取文件时，每次调用read命令都会读取文件中的"一行"文本,当文件没有可读的行时候，read命令将以非零状态退出

```bash
cat data.txt | while read line
do 
    echo $line
done

# or

while read line 
do
    echo "file:$line"
done < data.txt 
```

## 优化脚本分析

### 第一段

<font color="191970">cat > /etc/locale.conf << EOF 
LANG=en_US.utf8
LC_CTYPE=en_US.utf8
EOF</font>


```text
cat > /etc/locale.conf    向locale.conf文件写入文本(覆盖)  其实就是清空再写入
<< EOF ... EOF 标记一段文字  
```

#### eof
在写脚本的时候，如果我们要向一个文件中写入多行内容，这个时候如果我们一直去使用echo追加的话会很麻烦，这个时候我们就要用到eof来隔离出一段多行的文本内容
```text
1.向文件中输入内容(覆盖)
cat > xxxx.md <<EOF
something
EOF
2.向文件中追加内容
cat >> xxx.md <<EOF
something
EOF
3.注释
:<<!
somthing 
!

另外EOF其实可以自定义，这样会显得很高逼格！
cat > xxx.md <<aaa
something 
aaa

```

----
### 第二段

<font color="191970">echo -e 'nameserver 192.168.147.20\nnameserver 192.168.21.20\nnameserver 114.114.114.114' > /etc/resolv.conf</font>

```text
将内容写到 /etc/resolv.conf 文件中(覆盖)
echo -e 在附加的过程中使用转义
```

#### echo
echo其实不仅仅具有print输出的作用,还可以在后面家执行的命令
echo [Option] [String]
```
-n 表述输出不换行
-e 启用反斜杠转义
-E 禁用反斜杠转义 (缺省)
\a 发出警报声

```
------
### 第三段
<font color="191970">KERNEL="kernel-ml"
HOST="OPS-HGH-FDI-020"
ZONE="Asia/Shanghai"
</font>
就是声明变量

-----


### 第四段
<font color="191970">

hostnamectl set-hostname \$HOST
echo \$HOST > /etc/hostname

</font>
 
修改所有主机名 ，变量为 host
将修改后的主机名 写入 /etc/hostname 

#### hostnamectl 
用来管理给定主机中使用的三种类型的主机名

- hostnamectl status  查看所有当前主机名
- hostnamectl set-hostname name 设定所有主机名
- hostnamectl set-hostname name [static | pretty | transient] 设定特定的主机名
- hostnamectl set-hostname "" [static | pretty | transient] 清除特定的主机名

----


### 第五段
timedatectl set-timezone $ZONE
timedatectl set-ntp 0
timedatectl set-local-rtc 0

将时区设置为  Asia/Shanghai
自动同步系统时间到上海时间

#### timedatectrl
timedatectrl可以查询和修改系统时钟和设置

- timedatectl status 显示当前时间和日期
- timedatectl list-timezones 查看所有可用的时区
- timedatectl set-timezone "Asia/Shanghai"  设置本地时区
- timedatectl set-timezone "UTC" 　推荐使用和设置协调世界时，即UTC。
- timedatectl set-time 15:00:00 设置时间
- timedatectl set-ntp true 自动同步时间到远程NTP服务器    1
- timedatectl set-ntp false 禁用NTP时间同步 0
- timedatectl set-local-rtc 0  将你的硬件时间设置为世界时   ///  timedatectl set-local-rtc 1 将你的硬件时间设置为本地时间

-----

### 第六段
set enforce 0
sed -r -i '/^SELINUX=/s^=.*^=disabled^g' /etc/selinux/config
sed -r -i '/\^[^root]/s:/bin/bash:/sbin/nologin:g' /etc/passwd
sed -r -i '/#Port 22/s^.*^Port 65422^g;/^PasswordAuthentication/s^yes^no^g' /etc/ssh/sshd_config
sed -r -i -e '/DefaultLimitCORE/s^.*^DefaultLimitCORE=infinity^g' -e '/DefaultLimitNOFILE/s^.*^DefaultLimitNOFILE=100000^g' -e '/DefaultLimitNPROC/s^.*^DefaultLimitNPROC=100000^g' /etc/systemd/system.conf

<!-- <font style='blue'>这里为什么要先设置成宽容模式，然后在将selinux改成disabled</font> -->
<!-- <font style='blue'>命令行里面是setenforce 0  而脚本里面是 set enforce 0</font> -->

```text
set enforce 0 临时设置selinux的运行模式为宽容模式
sed -r -i 使用正则并且修改文件 
'/^SELINUX=/s^=.*^=disabled^g' /etc/selinux/config 
进入到/etc/selinux/config配置文件下，匹配到以SELINUX开头的行 将=后面所有字符都替换成disabled
'/^[^root]/s:/bin/bash:/sbin/nologin:g' /etc/passwd 
进入到/etc/passwd 配置文件下，匹配到以root任意单词为开头的行 将/bin/bash替换成/sbin/nologin
'/#Port 22/s^.*^Port 65422^g;/^PasswordAuthentication/s^yes^no^g' 
进入到 /etc/ssh/sshd_config 配置文件下，匹配以#Port 22开头的内容，将他替换成Port 65422  ; 匹配到PasswordAuthentication开头的行将yes替换后成no
'/DefaultLimitCORE/s^.*^DefaultLimitCORE=infinity^g' -e '/DefaultLimitNOFILE/s^.*^DefaultLimitNOFILE=100000^g' -e '/DefaultLimitNPROC/s^.*^DefaultLimitNPROC=100000^g' /etc/systemd/system.conf  匹配到DefaultLimitCORE的行 将他所有的内容替换成DefaultLinmitCORE=infinity 多个替换
进入到 /etc/systemd/system.conf配置文件下 多个参赛进行替换
```

---------------


### 第七段
localectl set-locale LANG=en_US.UTF8
cat > /etc/locale.conf <<EOF
LANG=en_US.utf8
LC_CTYPE=en_US.utf8
EOF

设置区域语言为英语
设置语言符号为英语

--------------------

### 第八段

cat > /etc/security/limits.d/20-nproc.conf  <<EOF
*          soft    nproc    10240
root       soft    proc     unlimited
EOF

将内容写入到/etc/security/limits/20-nproc.conf文件下 (覆盖写入)

-------------------------------------

### 第九段
cat > /etc/cron.d/upyun <<EOF
CRON_TZ=$ZONE
0 * * * * root (/usr/sbin/ntpdate -o3 192.168.147.20 211.115.194.21 133.100.11.8 142.3.100.15)
EOF
ntpdate -o3 192.168.147.20 211.115.194.21 133.100.11.8 142.3.100.15

将内容写入到/etc/cron.d/upyun 文件下(覆盖写入)

-----------------------------------------

### 第九段
yum install -y tree ntpdate telnet bc nc net-tools wget lsof rsync bash-completion iptables-services bind-utils python-setuptools yum-utils epel-release

<details>
<summary>配置摘要</summary>
```
yum 是rpm软件管理器
tree 目录树指令
ntpdate 
telnet
bc
nc
net-tools
wget
lsof
rsync
bash-completion
iptables-services
bind-utils
python-setuptools
yum-utils
epel-release
```
</details>

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum makecache fast
yum --enablerepo=elrepo-kernel -y install docker-ce

```
看不懂
看不懂
添加docker-ce 镜像到yum源
看不懂
看不懂
```

----------------------------------------------

### 第十段

systemctl disable firewalld postfix auditd irqbalance tuned
\#systemctl unmask NetworkManager
\#systemctl enable NetworkManager
sed -r -i 's/biosdevname=0//g;s/net.ifnames=0//g' /etc/sysconfig/grub
sed -r -i 's/biosdevname=0//g;s/net.ifnames=0//g' /etc/default/grub
$(ip link|grep -iq "eth[0-9]\{1,3\}:.*up") && sed -r -i '/GRUB_CMDLINE_LINUX/s^(.*)="(.*)"$^\1="\2 biosdevname=0 net.ifnames=0"^g' /etc/sysconfig/grub
$(ip link|grep -iq "eth[0-9]\{1,3\}:.*up") && sed -r -i '/GRUB_CMDLINE_LINUX/s^(.*)="(.*)"$^\1="\2 biosdevname=0 net.ifnames=0"^g' /etc/default/grub


-------------------------
### 第十一段
if [ ! -z $KERNEL ];then
    yum --enablerepo=elrepo-kernel -y install docker-ce $KERNEL
    Version=`yum info $KERNEL|awk -F: '/Version/{print $2}'`
    Menu=`sed -r -n "s/^menuentry '(.*)' --class.*/\1/p" /boot/grub2/grub.cfg|grep $Version`
    grub2-set-default "$Menu"
    grub2-mkconfig -o /boot/grub2/grub.cfg
fi

-----------------------------------



### 最终版Centos配置
```shell
## centos优化脚本.sh

#!/bin/sh

:<<!
向DNS客户机中添加配置文件
!
echo -e 'nameserver 192.168.147.20\nnameserver 192.168.21.20\nnameserver 114.114.114.114' > /etc/resolv.conf

:<<!
声明变量
!

KERNEL="kernel-ml"
HOST="OPS-HGH-FDI-020"
ZONE="Asia/Shanghai"

hostnamectl set-hostname $HOST
echo $HOST > /etc/hostname

timedatectl set-timezone $ZONE
timedatectl set-ntp 0
timedatectl set-local-rtc 0

set enforce 0
sed -r -i  '/^SELINUX=/s^=.*^=disabled^g' /etc/selinux/config
sed -r -i '/^[^root]/s:/bin/bash:/sbin/nologin:g' /etc/passwd
sed -r -i '/#Port 22/s^.*^Port 65422^g;/^PasswordAuthentication/s^yes^no^g' /etc/ssh/sshd_config
sed -r -i -e '/DefaultLimitCORE/s^.*^DefaultLimitCORE=infinity^g' -e '/DefaultLimitNOFILE/s^.*^DefaultLimitNOFILE=100000^g' -e '/DefaultLimitNPROC/s^.*^DefaultLimitNPROC=100000^g' /etc/systemd/system.conf

localectl set-locale LANG=en_US.UTF8
cat > /etc/locale.conf <<EOF
LANG=en_US.utf8
LC_CTYPE=en_US.utf8
EOF

cat > /etc/security/limits.d/20-nproc.conf  <<EOF
*          soft    nproc    10240
root       soft    proc     unlimited
EOF

cat > /etc/cron.d/upyun <<EOF
CRON_TZ=$ZONE
0 * * * * root (/usr/sbin/ntpdate -o3 192.168.147.20 211.115.194.21 133.100.11.8 142.3.100.15)
EOF
ntpdate -o3 192.168.147.20 211.115.194.21 133.100.11.8 142.3.100.15

yum install -y tree ntpdate telnet bc nc net-tools wget lsof rsync bash-completion iptables-services bind-utils python-setuptools yum-utils epel-release
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum makecache fast
yum --enablerepo=elrepo-kernel -y install docker-ce

systemctl disable firewalld postfix auditd irqbalance tuned
#systemctl unmask NetworkManager
#systemctl enable NetworkManager
sed -r -i 's/biosdevname=0//g;s/net.ifnames=0//g' /etc/sysconfig/grub                                                  
sed -r -i 's/biosdevname=0//g;s/net.ifnames=0//g' /etc/default/grub
$(ip link|grep -iq "eth[0-9]\{1,3\}:.*up") && sed -r -i '/GRUB_CMDLINE_LINUX/s^(.*)="(.*)"$^\1="\2 biosdevname=0 net.ifnames=0"^g' /etc/sysconfig/grub
$(ip link|grep -iq "eth[0-9]\{1,3\}:.*up") && sed -r -i '/GRUB_CMDLINE_LINUX/s^(.*)="(.*)"$^\1="\2 biosdevname=0 net.ifnames=0"^g' /etc/default/grub

if [ ! -z $KERNEL ];then
    yum --enablerepo=elrepo-kernel -y install docker-ce $KERNEL
    Version=`yum info $KERNEL|awk -F: '/Version/{print $2}'`
    Menu=`sed -r -n "s/^menuentry '(.*)' --class.*/\1/p" /boot/grub2/grub.cfg|grep $Version`                           
    grub2-set-default "$Menu"
    grub2-mkconfig -o /boot/grub2/grub.cfg
fi



```








<!-- more -->

