# 备份命令 dd
用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。

dd (参数)
```bash
if=文件名 #指定标准输入的文件
of=文件名 #指定输出文件名
ibs=bytes: # 一次读入bytes个字节，即指定一个块大小为bytes个字节。 
obs=bytes: # 一次读入bytes个字节，即指定一个块大小为bytes个字节。
bs=bytes:  #同时设置读入/输出的块大小为bytes个字节。
cbs=bytes:
skip=blocks:
seek=blocks:
count=blocks:
```


## 实战
```bash 
# 将本地的/dev/hdb整盘备份到/dev/hdd
dd if=/dev/hdb of=/def/hdd
# 将盘数据备份到文件 
dd if=/dev/sdb of=/root/image
# 备份恢复
dd if=/root/image of=/dev/sdb
# 压缩后保存
dd if=/dev/sdb  | gzip > /root/image.gz
# 
```
https://linux.cn/article-1429-1.html
http://www.markjour.com/article/linux-clean-disk.html