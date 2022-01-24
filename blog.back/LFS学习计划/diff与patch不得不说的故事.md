# diff与path不得不说的故事

很早以前 linux的bug修改是由各个研发者之间通过磁盘的形式进行交换更迭的，这样会变得很麻烦，于是后来有了写了patch，即补丁的形式进行内容更迭，为了产生补丁，也就是新旧文件的不同之处，产生了diff

如果你hack了开源代码，为了方便分享(如提交了Bug) 或自己留存使用，一般都要制作一个补丁。

综上，在linux补丁的制作过程中，我们需要用到两个命令，diff与patch


## 命令简介
**diff**
比较两个东西，并可同时记录下二者的区别。制作补丁时的一般用法
```bash
diss [option] source-file  destination-file
```
对于diff命令来说，他能够很好的比较文件之间的区别，包括文件夹中整个文件的对比，他有三种输出模式  normal context unified

normal 默认输出方式，不需要加任何参数
context 输出修改部分的上下文，默认前后3行  -c
unified 上下文合并后输出，默认前后3行 -u

-b --ignore-space-change 不检查空格字符的不同
-B --ignore-blak-lines 不检查空白行
-H --forward-ed 比较大文件时，加快速度
-i --ignore-case 不检查大小写的不同

-q 仅显示有无差异，不显示详细的信息
-w 忽略全部的空格字符


-r 递归。设置后diff会将两个不同版本源代码目录中的所有对应文件全部进行一次比较，包括子目录文件
-N 确保补丁文件将正确的处理已经创建或删除文件的情况
-un 输出每个修改前后的n行，-u默认3行

```bash
[root@gogogo alpaca]# diff -u tt.json yy.json
--- tt.json     2019-11-06 10:02:23.018769482 +0800
+++ yy.json     2019-11-06 10:02:45.362289254 +0800
@@ -1,4 +1,5 @@
 xxxxxxxxxxxxxxxxxxxxxx
+yyyyyyyyyyyyyyyyyyyyyy
 xxxxxxxxxxxxxxxxxxxxxx
 xxxxxxxxxxxxxxxxxxxxxx
 xxxxxxxxxxxxxxxxxxxxxx
[root@gogogo alpaca]#
```
如上所示，
---  tt.json是修改前文件  +++ yy.json 是修改后文件
@@ -1,4 是修改前文件的4行上下文
+1,5 是修改后文件的5行上下文
+表示新增
-表示删除
不带符号表示未改变 合并输出


**patch**
将diff记录的结果应用到相应文件夹上。

```bash
patch [option] [originalfile] [pathfile]
```

-pNum 忽略几层文件夹
-E 发现空文件则删除
-R 取消打过的补丁
-i 指定补丁文件
-o 输出到一个文件而不是覆盖



## 尝试打一次补丁
http://linux-wiki.cn/wiki/zh-hans/%E8%A1%A5%E4%B8%81(patch)%E7%9A%84%E5%88%B6%E4%BD%9C%E4%B8%8E%E5%BA%94%E7%94%A8