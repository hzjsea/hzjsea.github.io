# 鸟叔的linux私房菜
https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content/168.html
# grub介绍 
https://www.cnblogs.com/f-ck-need-u/p/7094693.html
# 分区
https://www.cnblogs.com/xiaoluo501395377/archive/2013/04/03/2997098.html

# linux启动程序
https://www.centos.bz/2017/08/centos7-boot-process-systemd-intro/
https://www.cnblogs.com/JMLiu/p/10183948.html


# linux的启动程序 grub介绍
之前两个步骤结束之后，linux会进入启动盘的第一个分区寻找MBR引导记录，首先这个记录是由512个字节组成，其中最前面的446个自己就是bootloader
在CENTOS7中，bootloader的主流为grub2.cfg 目录为/boot/grub2/grub.cfg
在之前的一些老版本中，使用的是grub.cfg 目录为/boot/grub/grub.cfg 
grub2.cfg相对于grub.cfg来说已经有了较大的改进,推荐使用grub2.cfg



## 流程以及功能
当BIOS读取完信息之后进入bootloader的流程，这个 boot loader 可以具有菜单功能、直接载入核心文件以及控制权移交的功能等， 
系统必须要有 loader 才有办法载入该操作系统的核心.

<font color='red'>bootloader载入主要分为两部分</font>

第一步:
执行 boot loader 主程序： 第一阶段为执行 boot loader 的主程序，这个主程序必须要被安装在开机区,由于MBR中的bootloader仅仅只占用了446字节的内存，直接启动整个程序是不可能的，因此他会先开启一个主程序，这446的字节也只是安装了这个主程序

第二步:
执行主程序载入配置文件，通过 boot loader 载入所有配置文件与相关的环境参数文件 （包括文件系统定义与主要配置文件 grub.cfg）

```bash
[root@study ~]# ls -l /boot/grub2
-rw-r--r--.  device.map            # grub2 的设备对应档（下面会谈到）
drwxr-xr-x.  fonts                 # 开机过程中的画面会使用到的字体数据
-rw-r--r--.  grub.cfg              # grub2 的主配置文件！相当重要！
-rw-r--r--.  grubenv               # 一些环境区块的符号
drwxr-xr-x.  i386-pc               # 针对一般 x86 PC 所需要的 grub2 的相关模块
drwxr-xr-x.  locale                # 就是语系相关的数据啰
drwxr-xr-x.  themes                # 一些开机主题画面数据

[root@study ~]# ls -l /boot/grub2/i386-pc
-rw-r--r--.  acpi.mod              # 电源管理有关的模块
-rw-r--r--.  ata.mod               # 磁盘有关的模块
-rw-r--r--.  chain.mod             # 进行 loader 控制权移交的相关模块
-rw-r--r--.  command.lst           # 一些指令相关性的列表
-rw-r--r--.  efiemu32.o            # 下面几个则是与 uefi BIOS 相关的模块
-rw-r--r--.  efiemu64.o
-rw-r--r--.  efiemu.mod
-rw-r--r--.  ext2.mod              # EXT 文件系统家族相关模块
-rw-r--r--.  fat.mod               # FAT 文件系统模块
-rw-r--r--.  gcry_sha256.mod       # 常见的加密模块
-rw-r--r--.  gcry_sha512.mod
-rw-r--r--.  iso9660.mod           # 光盘文件系统模块
-rw-r--r--.  lvm.mod               # LVM 文件系统模块
-rw-r--r--.  mdraid09.mod          # 软件磁盘阵列模块
-rw-r--r--.  minix.mod             # MINIX 相关文件系统模块
-rw-r--r--.  msdospart.mod         # 一般 MBR 分区表
-rw-r--r--.  part_gpt.mod          # GPT 分区表
-rw-r--r--.  part_msdos.mod        # MBR 分区表
-rw-r--r--.  scsi.mod              # SCSI 相关模块
-rw-r--r--.  usb_keyboard.mod      # 下面两个为 USB 相关模块
-rw-r--r--.  usb.mod
-rw-r--r--.  vga.mod               # VGA 显卡相关模块
-rw-r--r--.  xfs.mod               # XFS 文件系统模块 
```

## 主程序载入配置
bootloader载入配置是一步很重要的工作，他决定了你是否能够顺利开机

<font color='red'>从磁盘当中载入核心文件</font> 
```bash
（hd0,1）         # 一般的默认语法，由 grub2 自动判断分区格式
（hd0,msdos1）    # 此磁盘的分区为传统的 MBR 模式
（hd0,gpt1）      # 此磁盘的分区为 GPT 模式
```
其中hd0 表示第一块接收到的盘，第二个则为1 表示成hd1
而后面的1 msdos1 gpt1 则为分区号

例子: 第一块盘的第2个分区  (hd0,2)
例子: 搜寻到的第3块盘的第3个MBR分区 (hd2,msdos3)


<font color='green'>在旧版的grub中,grub.cfg的标示为 无论分区还是搜寻的盘，第一永远以0开始</font>

例题：假设你的系统仅有一颗 SATA 硬盘，请说明该硬盘的第一个逻辑分区在 Linux 与 grub2 当中的文件名与代号：答：因为是 SATA 磁盘，加上使用逻辑分区，因此 Linux 当中的文件名为 /dev/sda5 才对 （1~4 保留给 primary 与 extended 使用）。 至于 grub2 当中的磁盘代号则由于仅有一颗磁盘，因此代号会是“ （hd0,msdos5） ”或简易的写法“ （hd0,5） ”才对。

## grub2配置文件
主程序主要载入的是/boot/grub2/grub.cfg这个文件
```bash
set default #指定哪个选项开机
set timeout #指定默认秒数
menuentry # 菜单设置，如果有印象的话，在开机的时候是可以选择的
```
计时:
```bash 
if [ x$feature_timeout_style = xy ] ; then
  set timeout_style=menu
  set timeout=5
# Fallback normal timeout code in case the timeout_style feature is
# unavailable.
else
  set timeout=5
fi
```

如果确定选择，则直接运行程序，如果没有，则倒计时5秒
菜单:
```bash
menuentry 'CentOS Linux (0-rescue-f1c72b3b715d4670bfbbe460d57896dc) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-f1c72b3b715d4670bfbbe460d57896dc-advanced-add25f64-dd65-491b-9afe-2837908a644a' {
        load_video
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1'  add25f64-dd65-491b-9afe-2837908a644a
        else
          search --no-floppy --fs-uuid --set=root add25f64-dd65-491b-9afe-2837908a644a
        fi
        linux16 /boot/vmlinuz-0-rescue-f1c72b3b715d4670bfbbe460d57896dc root=UUID=add25f64-dd65-491b-9afe-2837908a644a ro crashkernel=auto rhgb quiet
        initrd16 /boot/initramfs-0-rescue-f1c72b3b715d4670bfbbe460d57896dc.img
} 
```
如果选择，则会载入{}中的内容


## 直接配置grub2.cfg

## 间接配置grub2.cfg
如果直接配置grub2.cfg会比较麻烦，内容太多不好改，我们可以修改/etc/default/grub
```bash
[root@study ~]# cat /etc/default/grub
GRUB_TIMEOUT=5                   # 指定默认倒数读秒的秒数
GRUB_DEFAULT=saved               # 指定默认由哪一个菜单来开机，默认开机菜单之意
GRUB_DISABLE_SUBMENU=true        # 是否要隐藏次菜单，通常是藏起来的好！
GRUB_TERMINAL_OUTPUT="console"   # 指定数据输出的终端机格式，默认是通过文字终端机
GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap crashkernel=auto rhgb quiet"
                                 # 就是在 menuentry 括号内的 linux16 项目后续的核心参数
GRUB_DISABLE_RECOVERY="true"     # 取消救援菜单的制作 
```


## 生成grub.cfg文件
根据/etc/default/grub的配置内容,生成配置文件
```bash
# 编写配置内容
[root@study ~]# vim /etc/default/grub
GRUB_TIMEOUT=40
GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=menu
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap crashkernel=auto rhgb
  quiet elevator=deadline"
GRUB_DISABLE_RECOVERY="true"

# 生成配置文件
grub2-mkconfig -o /boot/grub2/grub.cfg
# 他会去/etc/grub.d/寻找所要用到的内容

root:/etc/grub.d# ls -l /etc/grub.d/
total 72
-rwxr-xr-x. 1 root root  8702 Aug  8 19:31 00_header
-rwxr-xr-x. 1 root root  1043 Mar 22  2019 00_tuned
-rwxr-xr-x. 1 root root   232 Aug  8 19:31 01_users
-rwxr-xr-x. 1 root root 10781 Aug  8 19:31 10_linux
-rwxr-xr-x. 1 root root 10275 Aug  8 19:31 20_linux_xen
-rwxr-xr-x. 1 root root  2559 Aug  8 19:31 20_ppc_terminfo
-rwxr-xr-x. 1 root root 11169 Aug  8 19:31 30_os-prober
-rwxr-xr-x. 1 root root   214 Aug  8 19:31 40_custom
-rwxr-xr-x. 1 root root   216 Aug  8 19:31 41_custom
-rw-r--r--. 1 root root   483 Aug  8 19:31 README
```

- 00_header：主要在创建初始的显示项目，包括需要载入的模块分析、屏幕终端机的格式、倒数秒数、菜单是否需要隐藏等等，大部分在 /etc/default/grub 里面所设置的变量，大概都会在这个脚本当中被利用来重建 grub.cfg
- 10_linux：根据分析 /boot 下面的文件，尝试找到正确的 linux 核心与读取这个核心需要的文件系统模块与参数等，都在这个脚本运行后找到并设置到 grub.cfg 当中。 因为这个脚本会将所有在 /boot 下面的每一个核心文件都对应到一个菜单，因此核心文件数量越多，你的开机菜单项目就越多了。 如果未来你不想要旧的核心出现在菜单上，那可以通过移除旧核心来处理即可
- 30_os-prober：这个脚本默认会到系统上找其他的 partition 里面可能含有的操作系统，然后将该操作系统做成菜单来处理就是了。 如果你不想要让其他的操作系统被侦测到并拿来开机，那可以在 /etc/default/grub 里面加上“ GRUB_DISABLE_OS_PROBER=true ”取消这个文件的运行
- 40_custom：如果你还有其他想要自己手动加上去的菜单项目，或者是其他的需求，那么建议在这里补充即可

## 额外补充
如给开机启动的时候加上密码
https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content/168.html

## 额外补充2
<font color='red'>关于lfs构建中的问题</font>
lfs构建的过程中，最后一步会要求我们建立自己的grub文件，其中就需要指定搜寻盘与分区的位置
这里有两种解决方案

1. 直接在BIOS中设置开机启动为你写入lfs的那块盘，然后设置grub启动为(hd0,msdos3)
2. 不修改BIOS中的设置，默认开机选项为你的宿主系统，通过BIOS阶段后，进入menuentry选项，这个时候我们可以通过选择菜单进入到我们设置的盘。这个方案需要我们在宿主机中写入menuentry 如果觉得麻烦可以使用第一种

开机忘记了密码 参考一下链接
https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content/169.html