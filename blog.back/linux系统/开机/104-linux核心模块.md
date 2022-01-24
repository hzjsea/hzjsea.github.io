# 核心与核心模块

- 核心: /boot/vmlinuz或者/boot/vmlinuz-version
- 核心解压缩所需要 RAM Disk : /boot/initramfs (/boot/initramfs-version)
- 核心模块  /lib/modules/version/kernel 或者 /lib/modules/$(uname -r)/kernel
- 核心源代码 /usr/src/linux 或者 /usr/src/kernels

如果核心被顺利载入  ，则一下信息被记录
核心版本 /proc/version
系统核心功能 /proc/sys/kernel



## 模块依赖
/lib/modules/kernel下面的文件
```bash
arch    ：与硬件平台有关的项目，例如 CPU 的等级等等；
crypto    ：核心所支持的加密的技术，例如 md5 或者是 des 等等；
drivers    ：一些硬件的驱动程序，例如显卡、网卡、PCI 相关硬件等等；
fs    ：核心所支持的 filesystems ，例如 vfat, reiserfs, nfs 等等；
lib    ：一些函数库；
net    ：与网络有关的各项协定数据，还有防火墙模块 （net/ipv4/netfilter/*） 等等；
sound    ：与音效有关的各项模块
```
使用命令查看各个模块的依赖性
```bash
depmod [-Ane]
-A 不加参数 depmod会主动分析目前核心的模块，并重新写入到/lib/modules/modules.dep下面 ,加上-A就是会添加或修改新的模块
-n 不写入文件，直接输出

```

查看当前核心载入了多少模块  lsmod
查看模块详细信息  modinfo [adln] [module_name]
-a 列出作者名字
-d  列出modules说明
-l  列出授权
-n  列出模块详细路径


自行载入模块，不考虑依赖性  insmod [/full/path] [module_name]
删除模块 rmmod [-fw] module_name 

以上两个命令 如果模块之间有依赖，则无法成功执行

载入模块，考虑依赖性 modprobe [cfr] module_name
-c 列出目前所有模块
-f 强制载入模块
-r 移除模块