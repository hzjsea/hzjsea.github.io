# Linux下的压缩文件


## 不同操作系统下常用的压缩包
windows:
    .rar
    .zip
Linux:
    .gz gzip    //压缩工具压缩的文件
    .bz2 bzip2  //压缩工具压缩的文件
    .tar tar    //tar没有压缩功能，只是把一个目录合并成一个文件
    .tar.gz     //tar打包，然后用gzip压缩归档
    .tar.bz2    //tar打包，然后用bzip2压缩归档
    .tar.xz     //tar打包，然后用xz压缩归档
Linux常用压缩文件以.tar.gz结尾


## Linux下的压缩和解压缩工具
zip是压缩工具，unzip是解压缩工具
yum install zip unzip -y
压缩文件
zip filename.zip filename

zip:
-q:不显示指令执行过程
-m:文件压缩完成之后，删除源文件
-r:递归处理，将指定目录下的所有文件和子文件一并处理压缩

unzip:
-d<目录>: 指定文件解压缩后所要存储的目录
-q: 不显示指令执行过程

-                                                                                                        

```text
匹配mono为名的文件并压缩
zip mono.zip mono*
```

----------------------------------------------------------------------------------

tar 是一个归档工具，但加上不同的指令，也具有解压缩的能力
tar [-zjxcvfpP] filename
归档
c 创建新的归档文件
x 对归档文件解包
t 列出归档文件里的文件列表
归档压缩
f 指定包文件名，多参数时f写最后
z 使用gzip压缩归档后的文件(.tar.gz)
j 使用bzip2压缩归档后的文件(.tar.bz2)
J 使用xz压缩归档后的文件(tar.xz)
p 创建压缩归档文件时，保留源文件的权限
--exclude 在打包的时候写入需要排除的文件或目录

v 查看归档或解包的过程
C 指定解压目录的位置

```text
常用打包与压缩组合
<!-- czf 打包tar.gz格式 -->
<!-- cjf 打包tar.bz2格式 -->
<!-- zxf 解压tar.gz格式 -->
<!-- jxf 解压tar.bz2格式 -->
xf  解压压缩包(自动选择)
tf  查看压缩包内容(自动选择)
```

安装gzip,bzip2,xz软件包
yum install gzip bzip2 xz -y

```text
对文档进行归档
tar -cf test.tar mono*
使用gzip对文档进行归档后压缩
tar -czf test.tar.gz mono*
使用bzip2压缩归档后的文件
tar -cjf test.tar.bz2 mono*
使用xz压缩归档后的文件
tar -cJf test.tar.xz mono*
创建压缩文件，排除单个文件
tar -czf test.tar.gz --exclude=name.txt
创建压缩文件，排除多个文件需要创建一个文件来存储列表(每行一个文件)
tar -czf test.tar.gz --exclude=exclude.list
```
------------------------------------------------------------------------------