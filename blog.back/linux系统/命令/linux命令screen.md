# linux下的终端切换自由软件
GNU Screen是一款由GNU计划开发的用于命令行终端切换的自由软件。用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。
GNU Screen可以看作是窗口管理器的命令行界面版本。它提供了统一的管理多个会话的界面和相应的功能。

```bash
screen ls # 查看screen窗口列表
screen -d xx # 离线xx 窗口
screen -h <行数> # 指定视窗的缓冲区行数
screen -r xx # 恢复 xx 窗口
screen -x pid  # 恢复之前离线的窗口
screen -wipe # 检查目前所有的screen作业，并删除已经无法使用的screen作业
screen -S youname # 新建一个叫yourname的session

```
## screen 快捷键
```bash
c-a c #创建窗口
c-a w #窗口列表
c-a n #下一个窗口
c-a p #上一个窗口
c-a 0-9 #窗口切换
```

## mac使用screen连接交换机
screen /dev/tty.usb

## 问题
1. 出现问题
/dev/tty.usbserial' for R/W: resource busy".
是因为接口已经在用了，接口忙碌，使用lsof查看该接口链接状态

lsof | grep usbserial


➜  ~ lsof | grep usbserial
screen    54490 alpaca    5u  CHR  9,6    0t1564  679 /dev/tty.usbserial-AI06J4MD

## 根据进程号进入
screen -x 54490



