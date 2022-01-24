LFS学习笔记第一篇：准备篇

lfs 9.0版本文档 https://lctt.github.io/LFS-BOOK/
linux from scratch 官网： http://www.linuxfromscratch.org/
lfs 资源包下载地址 http://mirror.jaleco.com/lfs/pub/lfs/lfs-packages/lfs-packages-9.0.tar
查看自己的机有几个内核  more /proc/cpuinfo |grep "physical id"|wc -l器
cpu ：8核 版本 CentOS Linux release 7.7.1908 (Core)
宿主机系统分区：

lfs分区： /dev/sdd1 20GB 无swap分区


## 检查宿主机器中的软件

https://lctt.github.io/LFS-BOOK/lfs-systemd/chapter02/hostreqs.html
上面的链接列出一些需要的内容 比如 7.6系统下make默认为3.6版本，而他要求的是4.0版本
### make版本升级
```bash
cd /tmp
 
wget http://mirrors.ustc.edu.cn/gnu/make/make-4.0.tar.gz
tar xf make-4.0.tar.gz 
cd make-4.0/
./configure 
make
make install
make -v
# 此时的 make 还是3.82 与环境变量有关系
/usr/local/bin/make -v
# 这是我们刚安装的 make 它的版本是4.0
whereis make
# 找一下都有哪些 make
cd /usr/bin/
mv make make.bak
# 把默认的 make 改名 
ln -sv /usr/local/bin/make /usr/bin/make
# 建立一个软连接
make -v
```
编译最小系统主要地点是在宿主机器上完成，编译的过程中需要用到宿主机器上的一些软件 
为了防止破坏宿主机器上原有的内容需要做到环境隔离，这里隔离使用chroot


### 检查软件环境
```bash
mkdir lfs_make
cd lfs_make

cat > version-check.sh << "EOF"
#!/bin/bash
# Simple script to list version numbers of critical development tools
export LC_ALL=C
bash --version | head -n1 | cut -d" " -f2-4
echo "/bin/sh -> `readlink -f /bin/sh`"
echo -n "Binutils: "; ld --version | head -n1 | cut -d" " -f3-
bison --version | head -n1
if [ -h /usr/bin/yacc ]; then
echo "/usr/bin/yacc -> `readlink -f /usr/bin/yacc`";
elif [ -x /usr/bin/yacc ]; then
echo yacc is `/usr/bin/yacc --version | head -n1`
else
echo "yacc not found"
fi
bzip2 --version 2>&1 < /dev/null | head -n1 | cut -d" " -f1,6-
echo -n "Coreutils: "; chown --version | head -n1 | cut -d")" -f2
diff --version | head -n1
find --version | head -n1
gawk --version | head -n1
if [ -h /usr/bin/awk ]; then
echo "/usr/bin/awk -> `readlink -f /usr/bin/awk`";
elif [ -x /usr/bin/awk ]; then
echo yacc is `/usr/bin/awk --version | head -n1`
else
echo "awk not found"
fi
gcc --version | head -n1
g++ --version | head -n1
ldd --version | head -n1 | cut -d" " -f2- # glibc version
grep --version | head -n1
gzip --version | head -n1
cat /proc/version
m4 --version | head -n1
make --version | head -n1
patch --version | head -n1
echo Perl `perl -V:version`
sed --version | head -n1
tar --version | head -n1
makeinfo --version | head -n1
xz --version | head -n1
echo 'main(){}' > dummy.c && g++ -o dummy dummy.c
if [ -x dummy ]
then echo "g++ compilation OK";
else echo "g++ compilation failed"; fi
rm -f dummy.c dummy
EOF

sh version-check.sh
```
如果出现了一些错误，则需要安装对应的软件，这是必须的
其中makeinfo == yum install texinfo
    g++ == yum install gcc-c++
    gcc == yum install gcc
 version-check.sh: not found == yum install bzip2 也不知道为什么没有显示出软件包的名字
    yacc not found == ln -s /usr/bin/bison /usr/bin/yacc  网上是说下载byacc 但是这里软连接过来也没问题
[root@test-ceph ~]# yum search yacc
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * centos-sclo-rh: mirrors.163.com
 * centos-sclo-sclo: mirrors.163.com
 * elrepo: mirror.rackspace.com
 * epel: mirrors.aliyun.com
============================================================================================================================== N/S matched: yacc ===============================================================================================================================
byacc.x86_64 : Berkeley Yacc, a parser generator
byaccj.x86_64 : Parser Generator with Java Extension
php-symfony-property-access.noarch : Symfony PropertyAccess Component
python-ply.noarch : Python Lex-Yacc
python36-ply.noarch : Python Lex-Yacc

  名称和简介匹配 only，使用“search all”试试。
yum install byacc
```
makeinfo 缺失的话就  yum install texinfo


**第二步检测时候出现如下错误**
```bash
[root@test-ceph lfs]# sh library-check.sh
libgmp.la: not found
libmpfr.la: not found
libmpc.la: not found 
```
https://unix.stackexchange.com/questions/135031/linux-from-scratch-libgmp-la-libmpfr-la-and-libmpc-la-not-found-during-versio
https://www.linuxquestions.org/questions/linux-from-scratch-13/libgmp-la-not-found-4175520439/ 这里说到如果你同时缺少了这三个库，那么你不需要担心，因为这是正常的。
但是如果你用了了其中的两个或者一个，那就是有问题了

### 系统隔离

分区
```bash
fdisk /dev/sdc
n --> p --> 1 --> Enter --> +20G --> w --> p


   设备 Boot      Start         End      Blocks   Id  System
/dev/sdc1            2048    41945087    20971520   83  Linux
```
分区可启动
```bash 
fdisk /dev/sdc
a --> 1 --> w
```

制作文件系统
mkfs -v ext4 /dev/sdc1

挂载分区
mkdir /opt/lfs
mount /dev/sdc /opt/lfs

### 准备编译环境

#### 下载安装包
在文档中 会用$LFS来表明一个路径，但我觉得不好用。。
```bash 
export LFS=/opt/lfs  # 声明变量 如果需要重启不变，可以在.brashrc中设置
mkdir -v $LFS/sources # 创建资源文件夹 主要用来存放安装包和补丁
chmod -v a+wt $LFS/sources # 更改权限  -->
wget https://linux.cn/lfs/LFS-BOOK-7.7-systemd/wget-list  # 获取包列表，这里是国外的
# 国内的建议使用 https://github.com/LCTT/LFS-BOOK-7.7-systemd/edit/master/wget-list-LFS7.7-systemd-USTC
wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
# 根据安装包列表 下载到资源文件夹 并使用断点传序的方式下载，即中途断开可以继续下载
```
#### 创建tool目录
这个目录主要是用来存放整个lfs的环境，即到时候我们会进入资源文件夹进行压缩和编译，将编译出来的内容放到/opt/lfs/tools下面。想要了解整个过程，可以在每次编译完一个包后查看tools中的变化

```bash
mkdir -v /opt/lfs/tools
ln -sv /opt/lfs/tools / # 将tools映射到根目录下面 这里需要注意的是 扩展链接。只要不删除/opt/lfs下面的/tools文件夹 就没事了 
```
### 用户环境
为了不污染宿主系统的内容，我们会创建一个用户来作为操作者，在安装之前所有的编译工作都教给这个操作者即lfs
```bash
groupadd lfs #添加用户组
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
passwd lfs 
123
chown -v lfs.lfs $LFS/tools   # 更改文件权限
# changed ownership of "/tools" from root:root to lfs:lfs   
chown -v lfs.lfs $LFS/sources # 更改文件权限
# changed ownership of "/sources" from root:root to lfs:lfs
su - lfs # 切换lfs用户并跳到指定给lfs的家目录 

# 设置.bash_profile
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF

# 设置.bashrc
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/opt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/tools/bin:/bin:/usr/bin
export LFS LC_ALL LFS_TGT PATH
export MAKEFLAGS='-j 4'        #四核机器，设置编译时使用4个cpu，加快编译速度
EOF

# 如果你不清楚你的机器是几核的
more /proc/cpuinfo |grep "physical id"|wc -l
```
