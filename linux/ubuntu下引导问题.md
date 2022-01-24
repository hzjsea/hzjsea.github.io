---
title: ubuntu下引导问题
date: 2021-04-26 10:28:18
categories:  [ubuntu]
tags: [ubuntu]
---


<!--more-->


# ubuntu下引导问题


ubuntu下引导无法启动的问题
https://www.zybuluo.com/omg-two/note/621225

> error：unknown filesystem的解决办法

```bash

#查看当前分区情况
grub rescue>ls
# 查看当前分区配置, 然后会列出当前grub的设置，例如prefix=(hd0,msdos2)/boot/grub,root=hd0,msdos2等等，因为这个设置的错误，导致grub找不到正确的Ubuntu分区。
grub rescue>set
# 重新设置启动分区
grub rescue >set root=hd1,msdos5 
grub rescue> set prefix=(hd1,msdos5)/boot/grub
（有些情况下会是set prefix=(hd1,msdos5)/grub，以set之后显示的grub设置为依据）

# 判断是否为正确分区
insmod normal
# 重新进去系统
normal
# 修复分区
sudo update-grub
sudo grub-install /dev/sda
```

<div style='font-size:20px;color:red'>正常情况下，启动分区都是在/dev/sda下面的</div>

<div style='font-size:20px;color:green'>
手动分区,手动分区最好的方法是隔离 /boot下的内容，因为磁盘上是分扇区的， /boot区可能是只占了一小部分，而这一小部分基本上就是log 等等经常读写的内容，如果这块扇区坏了，还能保留其他内容 登录并修复

</div>   