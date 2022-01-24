---
title: linux磁盘管理
categories:
  - linux
tags:
  - linux
abbrlink: af2754a4
date: 2020-02-16 23:41:43
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux磁盘管理](#linux磁盘管理)
  - [磁盘磁盘结构以及磁盘分区](#磁盘磁盘结构以及磁盘分区)
  - [磁盘的分区](#磁盘的分区)
    - [管理磁盘分区](#管理磁盘分区)
    - [对磁盘进行分区](#对磁盘进行分区)
  - [linux文件系统](#linux文件系统)
    - [创建文件系统(格式化)](#创建文件系统格式化)
    - [操作文件系统](#操作文件系统)
    - [软连接与硬链接](#软连接与硬链接)
  - [设备挂载与卸载](#设备挂载与卸载)
  - [分区的自动挂载](#分区的自动挂载)
    - [实例](#实例)
  - [特殊设备loop挂载](#特殊设备loop挂载)
  - [注释](#注释)

<!-- /code_chunk_output -->
<!-- more -->
# linux磁盘管理
主要记录的是centos系统从分区开始到文件系统的初始化再到挂载和卸载
1.磁盘的结构，什么是分区
2.管理分区,文件系统格式化
3.文件的挂载，以及自动挂载文件

------------------------------------

## 磁盘磁盘结构以及磁盘分区
机械硬盘(HHD): 1.传统磁盘 2 主要是由盘片，磁头以及盘片转轴组成 3 传输速度较慢
固态硬盘(SSD):  1. 固态电子存储芯片阵列而制成  2.传输速度快

相较于HDD,SSD 在防震抗摔、传输速率、功耗、重量、噪音上有明显优势，SSD 传输速率性能是HDD 的2倍
相较于SSD,HDD 在价格、容量、使用寿命上占有绝对优势。

**磁盘接口**
不同的磁盘接口在linux的命名也不一样， SATA盘在linux中的名称是sda-sdz IDE接口的硬盘在linux中的名称为hda-hdz

**常用语**
head 磁头
track 磁道
cylinder 柱面
sector 扇区 

**分区方式**
MBR(Master Boot Record )分区方式 主引导记录，分区不超过2T，分区工具用fdisk
GPT  支持128个分区 分区为2T以上的硬盘 分区工具用 gdisk



<font color='red'>磁盘的物理组成部分？</font>



## 磁盘的分区
linux下的分区有三种: 主分区，扩展分区和逻辑分区
其中主分区最多只能有四个,扩展分区最多只有一个，主分区加扩展分区最多有四个
![](http://img.noback.top/blog/img20191107/linux磁盘分区.png)
prime为主分区，extend为扩展分区，分区总数最多4个。扩展分区不能写入数据只能包含逻辑分区
逻辑分区可以写入数据和格式化
![](http://img.noback.top/blog/img20191107/linux磁盘分区2.png)
http://img.noback.top/blog/img20191107/linux
当我们已经创建了3个主分区和一个扩展分区之后，再次创建的时候会要求我们的扩展分区中创建逻辑分区。扩展分区是指一块被分成很多块区域的总和，但它其实并没有空间，只是一个统称。

<font color='blue'>扩展分区和主分区有什么区别和用处吗，为什么不直接规定能够创建4个主分区，不需要扩展分区</font>  



### 管理磁盘分区
lsblk命令的英文是“list block”，即用于列出所有存储设备的信息，而且还能显示他们之间的依赖关系，但是它不会列出RAM盘的信息。除了lsblk外，fdisk -l 以及 cat /proc/partitions 也都可以列出块设备，另外blkid 可以查看设备的uuid数据
```bash
lsblk [参数] 

-d 仅列出磁盘本身
-a 显示所有设备
-f 显示文件系统信息
-m 显示权限信息(rwx等)
-l 以列表格式显示 默认以目录树形式显示
-P 使用key="value"格式显示
-r 使用原始格式显示  
-t 使用拓扑结构显示

[root@test-ceph ~]# lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda               8:0    0 447.1G  0 disk
├─sda1            8:1    0   200M  0 part /boot/efi
├─sda2            8:2    0     1G  0 part /boot
└─sda3            8:3    0   446G  0 part
  ├─centos-root 253:0    0    50G  0 lvm  /
  ├─centos-swap 253:1    0   7.8G  0 lvm  [SWAP]
  └─centos-home 253:2    0 388.1G  0 lvm  /home
sdc               8:32   0   1.8T  0 disk
└─sdc1            8:33   0   500M  0 part
sdd               8:48   0   1.8T  0 disk
[root@test-ceph ~]# blkid
/dev/mapper/centos-root: UUID="2150883c-9314-4a74-be7e-9edf9f7ea5b7" TYPE="xfs"
/dev/sda3: UUID="g0LdqD-NzAO-dxtL-GfHa-Yo93-rx84-axOrFw" TYPE="LVM2_member" PARTUUID="8e5051e5-fb1c-4176-a3b2-15a60633aa42"
/dev/mapper/centos-home: UUID="936625a5-486e-4984-92d4-f2dae5f5bda7" TYPE="xfs"
/dev/sdd1: UUID="c1f2819c-4422-41bf-a0a0-5739732c236e" TYPE="ext4"
```

NAME: 设备文件名
MAJ:MIN 主要次要设备
RM 是否为可卸载设备 0为否 1为是
SIZE 容量
RO 是否为可读设备
TYPE 磁盘种类  disk磁盘  partition分区  rom只读存储器  lvm 逻辑分区
MOUNTPOINT 挂载点





### 对磁盘进行分区
磁盘分区主要使用到两个命令  分别是fdisk和gdisk

**fdisk**
fdisk是linux下硬盘的分区工具，但是fdisk只能划分小于2T的分区，如果大于2T则需要使用parted
```bash
[root@gogogo ~]# fdisk --help
用法：
 fdisk [选项] <磁盘>    更改分区表
 fdisk [选项] -l <磁盘> 列出分区表
 fdisk -s <分区>        给出分区大小(块数)

选项：
 -b <大小>             扇区大小(512、1024、2048或4096)
 -c[=<模式>]           兼容模式：“dos”或“nondos”(默认)
 -h                    打印此帮助文本
 -u[=<单位>]           显示单位：“cylinders”(柱面)或“sectors”(扇区，默认)
 -v                    打印程序版本
 -C <数字>             指定柱面数
 -H <数字>             指定磁头数
 -S <数字>             指定每个磁道的扇区数 
```

-l 后边不跟设备名会直接列出系统中所有的磁盘设备以及分区表，加上设备名会列出该设备的分区表
```bash
[root@gogogo ~]# fdisk -l

磁盘 /dev/vda：21.5 GB, 21474836480 字节，41943040 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x0000aebb

   设备 Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    41929649    20963801   83  Linux

磁盘 /dev/vdb：42.9 GB, 42949672960 字节，83886080 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节 
```
如上表示有两块磁盘，其中磁盘vda有一个分区
``` bash
[root@gogogo ~]# fdisk -l /dev/vda

磁盘 /dev/vda：21.5 GB, 21474836480 字节，41943040 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x0000aebb

   设备 Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    41929649    20963801   83  Linux
```


使用fdisk对小于2T的盘进行分区，fdisk 如果不加 “-l” 则进入另一个模式，在该模式下，可以对磁盘进行分区操作。
```bash
[root@gogogo ~]# fdisk /dev/vdb
欢迎使用 fdisk (util-linux 2.23.2)。
更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。
Device does not contain a recognized partition table
使用磁盘标识符 0xa813628c 创建新的 DOS 磁盘标签。
命令(输入 m 获取帮助)：
命令(输入 m 获取帮助)：m
   a   toggle a bootable flag   # 一个可引导的标识
   b   edit bsd disklabel       # 编辑bsd磁碟标签
   d   delete a partition   # 删除一个分区
   g   create a new empty GPT partition table # 创建一个新的空GPT分区表
   l   list known partition types  # 列出已知的分区类型
   m   print this menu # 打印这个菜单
   n   add a new partition # 增加一个新分区
   p   print the partition table  # 打印分区表
   q   quit without saving changes # 在没有保存更改的情况下退出
   t   change a partitions system id # 改变分区的系统id
```

**另外一种磁盘分区的方式  parted  gdisk**
https://blog.csdn.net/qq_44714603/article/details/88659996
https://www.cnblogs.com/huyuanblog/p/10120460.html





## linux文件系统
在linux中所有硬盘都可以被看成一个一个的文件，使用者可以像访问文件一样浏览整个硬盘中的内容。因此这是硬盘转变称为文件需要做的内容.
由于版本的更迭不断的更新.在Centos7中已经使用xfs作为默认的文件系统,而在这之前还有ext家族，对于xfs文件系统来说，我们使用到的格式化工具是mkfs这个指令，也就是我们说的make filesystem 对于ext家族来说常用的工具就是mke2fs这个指令,具体的内容你可以在man他们的时候出现
```bash
[root@test-ceph ~]# man mke2fs
MKE2FS(8)                                             System Manager's Manual                                             MKE2FS(8)
NAME
       mke2fs - create an ext2/ext3/ext4 filesystem

[root@test-ceph ~]# man mkfs       
MKFS(8)                                                System Administration                                                MKFS(8)

NAME
       mkfs - build a Linux filesystem
```

**查看支持的文件系统**
查看支持的文件系统：/lib/modules/`uname –r`/kernel/fs

```bash
[root@test-ceph ~]# cd /lib/modules/3.10.0-957.27.2.el7.x86_64/kernel/fs
[root@test-ceph fs]# ls
binfmt_misc.ko.xz  cifs    ext4     gfs2   mbcache.ko.xz  nls        udf
btrfs              cramfs  fat      isofs  nfs            overlayfs  xfs
cachefiles         dlm     fscache  jbd2   nfs_common     pstore
ceph               exofs   fuse     lockd  nfsd           squashfs 
```

**设备文件名**
![](http://img.noback.top/blog/img2019110717/linux磁盘管理3.png)
linux下所有硬件都是文件，dev目录下所有文件都是硬件
```bash
[root@test-ceph dev]# ls -al | grep sda
brw-rw----   1 root disk      8,   0 11月  8 00:26 sda
brw-rw----   1 root disk      8,   1 11月  8 00:26 sda1
brw-rw----   1 root disk      8,   2 11月  8 00:26 sda2
brw-rw----   1 root disk      8,   3 11月  8 00:26 sda3
```
sda1表示第一个SATA硬盘的第一个分区

### 创建文件系统(格式化)
文件系统的创建过程也就是对磁盘的格式化，由于不同系统之间所设置的文件属性/权限并不相同，为了能够顺利的存取这些磁盘中的数据，就要对这些磁盘进行格式化

磁盘的格式化一共分为两种，分别是低级格式化和高级格式化,其中低级格式化主要由厂商完成，也就是硬盘的磁道划分，而高级格式化由我们来完成，他又称逻辑格式化，是根据用户选定的文件系统（Linux的文件系统：EXT2/EXT3/EXT4(CentOS6.3默认)），在磁盘的特定区域写入特定数据，在分区中划出一片用于存放文件分配表/目录表等用于文件管理的磁盘空间。
假设磁盘是一个柜子，就是把一个柜子分为一个个等大的小个子，再EXT4中每个数据块默认大小是4KB，这时如果有个10KB的文件要写入，就需要分配三个数据块，多余的2KB不存数据，且这三个数据块分布在柜子里的随机小格子中而不是连续排列的。
磁盘碎片整理：把保存同一个文件的数据块尽量放在一起。
每个文件都有一个i节点号(inode号)，通过它可以建立inode列表，查找文件时通过寻找inode号在列表中找到文件条款，从而知道文件保存在哪几个数据块，从而打开数据块拼凑成完整文件。
格式化的目的不是清空数据，而是写入文件系统。

<font color='blue'>什么是文件系统</font>
较新的操作系统的文件数据除了文件实际内容外， 通常含有非常多的属性，例如 Linux 操作系统的文件权限（rwx）与文件属性（拥有者、群组、时间参数等）。 文件系统通常会将这两部份的数据分别存放在不同的区块，权限与属性放置到 inode 中，至于实际数据则放置到 data block 区块中。 另外，还有一个超级区块 （superblock） 会记录整个文件系统的整体信息，包括 inode 与 block 的总量、使用量、剩余量等。

在上面有说到三个内容，其中
- inode主要记录权限和属性，一个文件占用一个inode，同时记录此文件的数据所在的 block 号码，因此在实际使用的过程中，如果发现硬盘还有很多的余地，但是却不能写入数据，那又可能是inode的数量到达了上限，导致这个的原因可能是该磁盘中的文件小而多，如何改变可以看下面mke2fs的命令
- block 实际记录的文件内容，如果文件太大，则会占用过多的block
- superblock 记录此 filesystem 的整体信息，包括inode/block的总量、使用量、剩余量， 以及文件系统的格式与相关信息等


<font color='blue'>linux中文件系统的存储方式</font>
linux中的存储方式被称为索引式存储方式。首先是linux的文件系统，经过格式化之后，磁盘出现inode以及block，其中inode记录权限属性以及存储了数据的block的位置id，如果我们要寻找，只需要知道inode下存储数据的位置节点，就可以快速的读取的数据。
对比U盘的FAT格式来说，他并没有inode存在，FAT采用的存储方式是链式存储方式，也就是当前的block的位置id被上一个block记录，并且记录了下一个 block的位置，当我们寻找位于中间编号的block块时，都必须重头开始。
<font color='red'>什么是磁盘重组</font>
磁盘重组 需要磁盘重组的原因就是文件写入的 block 太过于离散了，此时文件读取的性能将会变的很差所致。 这个时候可以通过磁盘重组将同一个文件所属的 blocks 汇整在一起，这样数据的读取会比较容易啊！因此FAT 的文件系统需要三不五时的磁盘重组一下
<font color='red'>如何有利的安置block和inode ?</font>


**xfs文件系统**
```bash
mkfs.xfs [选项] [设备名称]
```
-b + block容量
-f 先强制格式化本身存在的文件系统
-i 指定inode大小
   size=数值   最小是256bytes 最大是2K 一般保留256就行了
   internal=[0|1] log设备是否为内置，默认1为内置


**ext家族文件系统**
mke2fs
在磁盘分区上创建ext2 ext3 ext4 文件系统，默认情况下创建ext2
当用man查询这四个命令的帮助文档时，您会发现我们看到了同一个帮助文档，这说明四个命令是一样的。mke2fs常用的选项

-b 分区时设定每个数据区块占用空间大小，目前支持1024,2048以及4096bytes每个块
-i 设定inode的大小
-N 设定inode的数量，默认的inode数量不够，就可以自定设置了。
-c 在格式化前先检测一下磁盘是否有问题,加上这个选项后速度执行速度会慢
-L 预设该分区的标签label
-t y用来指定什么类型的文件系统  比如mke2fs -t ext4 /dev/sdd3  相当于mke2fs.ext4 /dev/sdd3

e2label
用来查看或修改分区的标签
```bash
[root@test-ceph ~]# e2label /dev/sdb1
/hzj
[root@test-ceph ~]# e2label /dev/sdb1 Label
[root@test-ceph ~]# e2label /dev/sdb1
Label
[root@test-ceph ~]# e2label /dev/sdb1 /tt
[root@test-ceph ~]# e2label /dev/sdb1
/tt
```


### 操作文件系统
**df**
列出文件系统的整体磁盘使用量
```bash
df [选项] [文件]
```
```bash
[root@gogogo ~]# df
文件系统          1K-块    已用     可用 已用% 挂载点
devtmpfs         992496       0   992496    0% /dev
tmpfs           1019000       0  1019000    0% /dev/shm
tmpfs           1019000  106028   912972   11% /run
tmpfs           1019000       0  1019000    0% /sys/fs/cgroup
/dev/vda1      20953560 6057836 14895724   29% /
tmpfs            203800       0   203800    0% /run/user/0 
```
df默认显示块使用量，-i 显示inode信息而非块使用量
```bash
[root@gogogo ~]# df -i
文件系统          Inode 已用(I)  可用(I) 已用(I)% 挂载点
devtmpfs         248124     319   247805       1% /dev
tmpfs            254750       1   254749       1% /dev/shm
tmpfs            254750     413   254337       1% /run
tmpfs            254750      17   254733       1% /sys/fs/cgroup
/dev/vda1      20963776  112147 20851629       1% /
tmpfs            254750       1   254749       1% /run/user/0 
```

-h 选择合适的单位显示
-k 以k为单位显示
-m 以m为单位显示

https://cshihong.github.io/2018/06/20/Linux%E7%A3%81%E7%9B%98%E7%AE%A1%E7%90%86/

**du**
评估文件系统的磁盘使用两
```bash
du [-abckmsh] [文件或者目录名] 
```

-a(all) 全部文件与目录大小都列出来 ,默认情况下du 就是列出所有的目录
-b(bytes) 列出的值以byte为的单位输出
-k -m 分别以k和以m为单位。du默认单位k
-h 自动调节单位
-c 最后加总
-s 只列出总和

### 软连接与硬链接
同windows中的快捷键一样，linux中也存在着两种链接  软连接与硬链接
**硬链接 hard line**
```bash
[root@test-ceph alpaca]# ln /etc/xx .
[root@test-ceph alpaca]# ls
xx
[root@test-ceph alpaca]# ll -i /etc/xxx
100781634 -rw-r--r-- 2 root root 2 11月 11 19:26 /etc/xxx
[root@test-ceph alpaca]# ll -i xx
100781634 -rw-r--r-- 2 root root 2 11月 11 19:26 xx
```
在这之后你会发现当前目录下已经存在这/etc/xx的硬链接,且他们指向的inode节点序号是相同的。当我们输入cat xxx 或者cat /etc/xxx 的时候，首先他会先去目录中寻找对应的文件名，文件名与inode有对应表，再根据对应的表去寻找对应的内容，而硬链接的创建就是在目录上多添加了一条指向与/etc/xxx相同的inode节点序号，再由inode指向对应的内容，他是在目录层面的操作，因此当我们删掉了其中一个文件的时候，你会发现另外一个文件依旧能够访问，因为他删除这个操作仅仅只是把多条指向inode节点的线路慢慢减少到一条，对文件内容并没有任何的影响。
硬连接在创建的过程中，由于本身的inode节点和文间内容已经存在，所以不会增加任何内存，即使有，也很小小小小
**软链接**
```bash
[root@test-ceph alpaca]# ln -s /etc/xxx .
[root@test-ceph alpaca]# ls
xx  xxx
[root@test-ceph alpaca]# ll -al
lrwxrwxrwx   1 root root    8 11月 11 19:40 xxx -> /etc/xxx
[root@test-ceph alpaca]# ll -i
总用量 4
100781634 -rw-r--r-- 2 root root 2 11月 11 19:26 xx
35994196 lrwxrwxrwx 1 root root 8 11月 11 19:40 xxx -> /etc/xxx
[root@test-ceph alpaca]# ll -i /etc/xxx
100781634 -rw-r--r-- 2 root root 2 11月 11 19:26 /etc/xxx
```
当我们用软连接的时候，我们会发现他们的inode并不相同，当我们cat xxx的时候 同样的他会先去目录中查找inode,得到软连接后的xxx文件的inode后会发现他被指向了了/etc/xxx的inode，再由/etc/xxx的inode指向最后的/etc/xxx中的内容，他们是真正意义上的共用inode，当源文件被删除的时候，软链接也所指向的inode被删除，文件内容不可达

## 设备挂载与卸载
分区和格式化之后，你的磁盘已经是一个文件系统了，这个时候该怎么去使用呢？这就涉及到了磁盘的挂载和卸载，格式化的后的磁盘是一个块设备文件，类型为b
与挂载有关的目录
/mnt /dev

挂载：在linux操作系统中， 挂载是指将一个设备（通常是存储设备）挂接到一个已存在的目录上，在挂载某个分区前要先建立一个挂载点，而这个已存在目录就是我们说的挂载点，之后再把我们需要的块设备文件挂载上去。 我们要访问存储设备中的文件，必须将文件所在的分区挂载到一个已存在的目录上， 然后通过访问这个目录来访问存储设备。<font color='red'>这个目录可以不为空，但挂载后这个目录下以前的内容将不可用,取消挂载之后文件会出现</font>,一般情况下我们将挂载文件放置在/mnt 下面
https://www.cnblogs.com/zhang-jun-jie/p/9266810.html


**mount命令**
```bash
mount -a [选项]
 mount [选项] [--source] <源> | [--target] <目录>
 mount [选项] <源> <目录>
 mount <操作> <挂载点> [<目标>]

选项：
 -a, --all               挂载 fstab 中的所有文件系统
 -c, --no-canonicalize   不对路径规范化
 -f, --fake              空运行；跳过 mount(2) 系统调用
 -F, --fork              对每个设备禁用 fork(和 -a 选项一起使用)
 -T, --fstab <路径>      /etc/fstab 的替代文件
 -h, --help              显示此帮助并退出
 -i, --internal-only     不调用 mount.<类型> 助手程序
 -l, --show-labels       列出所有带有指定标签的挂载
 -n, --no-mtab           不写 /etc/mtab
 -o, --options <列表>    挂载选项列表，以英文逗号分隔
 -O, --test-opts <列表>  限制文件系统集合(和 -a 选项一起使用)
 -r, --read-only         以只读方式挂载文件系统(同 -o ro)
 -t, --types <列表>      限制文件系统类型集合
     --source <源>       指明源(路径、标签、uuid)
     --target <目标>     指明挂载点
 -v, --verbose           打印当前进行的操作
 -V, --version           显示版本信息并退出
 -w, --rw, --read-write  以读写方式挂载文件系统(默认)

 -h, --help     显示此帮助并退出
 -V, --version  输出版本信息并退出 
```


----------------

## 分区的自动挂载
**文件/etc/fstab**

<font color='blue'>
当系统启动的时候，系统会自动地从这个文件读取信息，并且会自动将此文件中指定的文件系统挂载到指定的目录。下面我来介绍如何在此文件下填写信息在这个文件中，规定了一些在系统启动的时候，可以自动挂载的分区</font>
```bash
[root@test-ceph ~]# cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Fri Aug 30 03:36:37 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=1803ca43-a22c-4626-9ee2-f8f585847536 /boot                   xfs     defaults        0 0
UUID=944B-59F3          /boot/efi               vfat    umask=0077,shortname=winnt 0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0 
```

1. 第一列就是分区的标识，可以写分区的label，也可以写uuid,也可以写分区名。
2. 第二列是挂载点;
3. 第三列是分区的格式
4. 第四列是mount的一些挂载参数 一般情况下都是default,但也有其他的
 async/sync 异步写入,磁盘和内存不同步，系统每隔一段时间把内存数据写入磁盘中,而sync则会同步内存和磁盘中的数据
 auto/noauto 开机自动挂载/不自动挂载
 default 按照大多数永久文件系统的缺省值挂载自定义
 ro 仅只读权限挂载
 rw 按可读可写权限挂载
 exec/noexec 允许/不允许文件可执行，但千万不要把根分区挂载为noexec
 user/nouser 允许/不允许root外的其他用户挂载分区
 suid/nosuid 允许/不允许分区有suid，一般设置nosuid
 userquota 启动使用者磁盘匹配额模式
 grquota 启动群组磁盘匹配模式
5. 数字表示是否被dump备份，1是 0否
6. 开机是否自检磁盘 1，2都表示检测，0表示不检测，在Redhat/CentOS中，这个1，2还有个说法，/ 分区必须设为1，而且整个fstab中只允许出现一个1，这里有一个优先级的说法。1比2优先级高，所以先检测1，然后再检测2，如果有多个分区需要开机检测那么都设置成2吧,1检测完了后会同时去检测2。

于是我们可以添加一行需要执行的分区，比如在/etc/fstab下我们添加一行/dev/sdd /test1 ext3 default 0 0 这样系统在下次开启的时候就会将这个文件挂载上了，当然我们也可以用mount -a来挂载fastab下面所有的磁盘分区
实战：
```bash
# 创建一个文件夹 并进入
mkdir /mnt/usb && cd /mnt/usb
# 挂载u盘
mount /dev/sad1 /mnt/usb
# 查看u盘内容
ls /mnt/usb
# 解挂
umount /dev/sda1 /mnt/usb
```


### 实例
描述一个从创建分区-->格式化-->挂载

创建分区
```bash
# 首先查看当前可用块
[root@test-ceph ~]# lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdd               8:48   0   1.8T  0 disk
└─sdd1            8:49   0   100G  0 part /mnt/usb

# 对块分区
root@test-ceph ~]# fdisk  /dev/sdd
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
分区号 (2-4，默认 2)：2
起始 扇区 (209717248-3907029167，默认为 209717248)：
将使用默认值 209717248
Last 扇区, +扇区 or +size{K,M,G} (209717248-3907029167，默认为 3907029167)：+100G
分区 2 已设置为 Linux 类型，大小设为 100 GiB
命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: 设备或资源忙.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
正在同步磁盘。

命令(输入 m 获取帮助)：p

磁盘 /dev/sdd：2000.4 GB, 2000398934016 字节，3907029168 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0xc39fa97e

   设备 Boot      Start         End      Blocks   Id  System
/dev/sdd1            2048   209717247   104857600   83  Linux
/dev/sdd2       209717248   419432447   104857600   83  Linux
```
格式化
```bash
[root@test-ceph ~]# mke2fs -t ext4 /dev/sdd1
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=
OS type: Linux
块大小=4096 (log=2)
分块大小=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
6553600 inodes, 26214400 blocks
1310720 blocks (5.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=2174746624
800 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: 完成
正在写入inode表: 完成
Creating journal (32768 blocks): 完成
Writing superblocks and filesystem accounting information: 完成
```
挂载
```bash
[root@test-ceph ~]# mount /dev/sdd1 /mnt/usb

[root@test-ceph ~]# lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdd               8:48   0   1.8T  0 disk
└─sdd1            8:49   0   100G  0 part /mnt/usb

[root@test-ceph ~]# df -T
文件系统                类型         1K-块      已用     可用 已用% 挂载点
/dev/sdd1               ext4     103080888     61464 97760160    1% /mnt/usb
```

## 特殊设备loop挂载

如何挂载光盘/DVD镜像文件-->来自鸟哥的linux私房菜
```bash
[root@study ~]# ll -h /tmp/CentOS-7.0-1406-x86_64-DVD.iso
-rw-r--r--. 1 root root 3.9G Jul  7  2014 /tmp/CentOS-7.0-1406-x86_64-DVD.iso
[root@study ~]# mkdir /data/centos_dvd
[root@study ~]# mount -o loop /tmp/CentOS-7.0-1406-x86_64-DVD.iso /data/centos_dvd
[root@study ~]# df /data/centos_dvd
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/loop0       4050860 4050860         0 100% /data/centos_dvd
# 就是这个项目！ .iso 镜像文件内的所有数据可以在 /data/centos_dvd 看到！
[root@study ~]# ll /data/centos_dvd
total 607
-rw-r--r--. 1  500  502     14 Jul  5  2014 CentOS_BuildTag &lt;==瞧！就是DVD的内容啊！
drwxr-xr-x. 3  500  502   2048 Jul  4  2014 EFI
-rw-r--r--. 1  500  502    611 Jul  5  2014 EULA
-rw-r--r--. 1  500  502  18009 Jul  5  2014 GPL
drwxr-xr-x. 3  500  502   2048 Jul  4  2014 images
.....（下面省略）.....

[root@study ~]# umount /data/centos_dvd/ 
```



## 注释
在正常情况下我们所创建的文件像 \root\hzj\test.txt  \home\hzj\test.txt的数据内容是存在于引导盘，也就是你装系统的那个盘的，而挂载的其他盘可以用来做存储的作用，先挂载后移动内容


