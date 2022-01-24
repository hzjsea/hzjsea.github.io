
# centos7启动流程
1. <font color='red'>开机自导</font>
centos7一开始并不会直接启动引导，之前的步骤都是由BIOS完成
首先插上电源按下开机键，计算机开始加电，主板上的BIOS或者UEFI基本输入输出程序开始对硬件进行检查，检查内存、CPU等等。这时候如果你的内存或者其他的问题出现了问题，会出现开不了机的问题。
<font color='blue'>比如内存氧化导致开不了机,或者驱动找不到如没有插键盘鼠标(驱动找不到的情况在windows中比较常见)</font>

2. <font color='red'>选择引导设备</font>
一切自检完成之后，系统进入BIOS环节，这其中有很多的配置，又一个则是选择引导顺序，这其中包括从硬盘启动，从U盘启动等等，均在BOOT OPTION菜单中,如果没有作出任何选择，则默认引导第一个设备

3. <font color='red'>Bootloader  引导加载器</font>
确定引导介质后开始从中装载引导程序 grub  这是一个微小的程序,其配置文件地点在/etc/grub2(在centos7中，主流的boot loader引导程序已经是grub2了，在早期则是grub1或LILO)
<font color='blue'>mbr引导记录大小为512字节，其中前446字节就是bootloader，用来引到用户选择要启动的系统或不同内核版本，把用户选定的内核装载到RAM中的特定空间中，解压，展开，然后把系统控制权交给内核</font>
grub2就是Linux中Bootloader程序，由于MBR记录限制，所以grub2分为两个部分。
- 在MBR引导记录中，大小为446字节，主要功能是引导启动介质的grub主体文件
- partition,/boot/grub[2],此为grub的主体

4. <font color='red'>加载内核</font>
引导处理完成之后，内核开始加载。
kernel开始初始化，探测可识别的硬件设备，加载硬件启动程序。以只读方式加载根文件系统
硬件驱动成功后，kernel主动调用systemd程序，并以default.target流程开启
- systemd 执行 sysinit.target 初始化系统以及basic.target 准备操作系统
- systemd 启动 multi-user.target 下的本机与服务器服务
- systemd 指定 multi-user.target 下的 /etc/rc.d/rc.local文件
- systemd 指定mylti-user.target 下的 getty.target 及登陆服务
- systemd 指定 graphical 需要的服务


## BIOS 、boot loader 、 kernel
- BIOS (Basic input Output System)
- MBR (Master Boot Record)
MBR：虽然分区表有传统 MBR 以及新式 GPT，不过 GPT 也有保留一块相容 MBR 的区块，总之mbr就是磁盘最前面可安装boot loader 的区块

BIOS是隔离与系统的存在，也就是说不如什么系统，机器开机的同时，BIOS都会起来。
由于不同系统的文件系统格式不同，为了除了这些系统的核心载入文件(load) BIOS会通过INT 13信道找到当前系统的load文件，这个文件就会区执行boot loader 引导加载


当我们借由 boot loader 的管理而开始读取核心文件后，接下来， Linux 就会将核心解压缩到内存当中， 并且利用核心的功能，开始测试与驱动各个周边设备，包括储存设备、CPU、网卡、声卡等等。 此时 Linux 核心会以自己的功能重新侦测一次硬件，而不一定会使用 BIOS 侦测到的硬件信息。也就是说，核心此时才开始接管 BIOS 后的工作了。 那么核心文件在哪里啊？一般来说，他会被放置到 /boot 里面，并且取名为 /boot/vmlinuz 才对，在lfs搭建的过程中，最后引导的制作也同样会遇到这个文件
```bash
[root@localhost ~]# ls --format=single-column -F /boot
config-3.10.0-229.el7.x86_64                &lt;==此版本核心被编译时选择的功能与模块配置文件
grub/                                       &lt;==旧版 grub1 ，不需要理会这目录了！
grub2/                                      &lt;==就是开机管理程序 grub2 相关数据目录
initramfs-0-rescue-309eb890d3d95ec7a.img    &lt;==下面几个为虚拟文件系统文件！这一个是用来救援的！
initramfs-3.10.0-229.el7.x86_64.img         &lt;==正常开机会用到的虚拟文件系统
initramfs-3.10.0-229.el7.x86_64kdump.img    &lt;==核心出问题时会用到的虚拟文件系统
System.map-3.10.0-229.el7.x86_64            &lt;==核心功能放置到内存位址的对应表
vmlinuz-0-rescue-309eb890d09543d95ec7a*     &lt;==救援用的核心文件
vmlinuz-3.10.0-229.el7.x86_64*              &lt;==就是核心文件啦！最重要者vmlinuz-5.2.8-lfs-9.0-systemd                   ==lfs9.0核心文件
```

问题是，核心根本不认识 SATA 磁盘，所以需要载入 SATA 磁盘的驱动程序， 否则根本就无法挂载根目录。但是 SATA 的驱动程序在 /lib/modules 内，你根本无法挂载根目录又怎么读取到 /lib/modules/ 内的驱动程序？是吧！非常的两难吧！在这个情况之下，你的 Linux 是无法顺利开机的！ 那怎办？没关系，我们可以通过虚拟文件系统来处理这个问题。

虚拟文件系统 --> 一般的命名为/boot/initrd 或 /boot/initramfs,他可以通过boot loader来载入到内存中，这个文件会被解压缩并且在内存当中仿真成一个根目录，且此仿真在内存当中的文件系统能够提供一个可执行的程序通过该程序来载入开机过程中所最需要的核心模块， 通常这些模块就是 USB, RAID, LVM, SCSI 等文件系统与磁盘接口的驱动程序啦！等载入完成后， 会帮助核心重新调用 systemd 来开始后续的正常开机流程。



