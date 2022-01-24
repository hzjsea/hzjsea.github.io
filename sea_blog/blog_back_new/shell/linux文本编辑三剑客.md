---
title: linux文本编辑三剑客
categories:
  - shell
tags:
  - shell
abbrlink: ec496181
date: 2020-01-29 22:09:25
---
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux文本编辑三剑客](#linux文本编辑三剑客)
  - [grep基础](#grep基础)
    - [参数](#参数)
    - [实例](#实例)
    - [正则表达式](#正则表达式)
    - [实例](#实例-1)
  - [sed基础](#sed基础)
    - [参数:](#参数-1)
    - [sed自定义匹配分隔符](#sed自定义匹配分隔符)
    - [sed在mac和linux上的不同使用](#sed在mac和linux上的不同使用)
    - [Command匹配段](#command匹配段)
    - [处理规则](#处理规则)
    - [处理方式](#处理方式)
    - [使用案例](#使用案例)
  - [awk基础](#awk基础)
    - [awk的工作流程](#awk的工作流程)
    - [awk中的分隔符](#awk中的分隔符)
    - [awk格式化输出](#awk格式化输出)
    - [awk的内建变量](#awk的内建变量)
      - [分割内建变量](#分割内建变量)
      - [内建环境变量](#内建环境变量)
      - [关系运算符](#关系运算符)
      - [自定义变量](#自定义变量)
    - [awk-options部分](#awk-options部分)
    - [awk-action部分](#awk-action部分)
    - [awk中的Pattern部分](#awk中的pattern部分)
    - [awk中的正则模式](#awk中的正则模式)
    - [awk中的流程控制](#awk中的流程控制)
    - [awk中的内置函数](#awk中的内置函数)
    - [awk中的数组](#awk中的数组)
    - [awk中的数组实例](#awk中的数组实例)
    - [awk中调用shell 命令 system()](#awk中调用shell-命令-system)
  - [错误](#错误)

<!-- /code_chunk_output -->


<!-- more -->

# linux文本编辑三剑客
linux文本编辑三剑客，awk、grep、sed是linux操作文本的三大利器，合称文本三剑客，也是必须掌握的linux命令之一。三者的功能都是处理文本，但侧重点各不相同，其中属awk功能最强大，但也最复杂。grep更适合单纯的查找或匹配文本，sed更适合编辑匹配到的文本，awk更适合格式化文本，对文本进行较复杂格式处理

关于"三剑客"的特长
什么是三剑客的特长=
- grep 更适合单纯的查找和匹配文本
- sed 更适合编辑匹配的文本
- awk 更适合格式化文本,对文本进行较复杂格式处理
## grep基础
grep（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来


### 参数
```bash
grep (选项) pattern flie


-A <显示行数> 除了显示符合匹配行，也显示匹配行后的n行
-B <显示行数> 除了显示符合匹配行，也显示匹配行前的n行
-C <显示行数> 除了显示符合匹配行，也显示匹配行前后的n行  但不显示每一行的行数
-c 仅显示统计匹配行数的总数
-e 实现多个选项件的逻辑关系 or
-E 扩展的正则表达式
-f FILE：从FILE获取匹配参数格式
-i --ignore-case 忽略字符大小写的差别
-n 输出匹配到内容后，重新按照顺序排序，从1开始
-o 仅显示匹配到的字符串
-v 反向匹配，显示不被匹配的内容
-w 严格匹配
-q 静默输出，也就是匹配结束之后返回状态码不输出内容
```

### 实例
```bash
# 内容准备
more  xx.txt
helloworld
helloworld
helloworld
xxxx

>> grep -w  "helloworld" xx.txt # 严格匹配
helloworld
helloworld
helloworld

>> grep -w  "helloworld" xx.txt -v # 先严格匹配，然后再反向
xxxx

>> grep -w  "helloworld" xx.txt -n
1:helloworld
2:helloworld
3:helloworld

```



### 正则表达式

**匹配内容**
.匹配全部字符，不匹配空行
[]匹配指定范围内的任意单个字符
[^] 取反
[:alnum:] [0-9a-zA-Z]
[:alpha:] [a-zA-Z]
[:upper:] 或 [A-Z]
[:lower:] 或 [a-z]
[:blank:] 匹配空白字符
[:punct:] 标点符号
**匹配次数**

```bash
*匹配任意次 0-n次 ,贪婪模式(尽可能多匹配)
.*匹配任意次 1-n次 
\?匹配 0-1次
\+至少匹配1次
```

**位置确定**
```bash
^ 行首
$ 行尾
^$ 空白行
```


**划组匹配**
在正则表达示中使用之前匹配到的内容，即划组匹配

```bash
# 分组形式 \(\)
# 读取方式 \1 \2 \3
```
![](http://noback.upyun.com/blog/img20191112/正则表达式.png)


### 实例 

```bash
#多文件查找
grep "match_pattern" file1 file2 file3 
#反项查找
grep -v "macth_pattern" file1 file2 file3 
#匹配项标记颜色
grep "macth_pattern" file1 --color=auto 
#正则匹配
grep -E "[1-9]+"
或
egrep "[1-9]+"
#只显示匹配到的字符串
echo this is a test line. | grep -o -E "[a-z]+\."
#统计匹配到的内容数量
grep -c "text" file1 
#显示匹配到的行号
grep -n "text" file1 
#多级目录中递归搜索
grep "text" . -r -n 
#忽略大小写
echo "hello world" | grep -i "HELLO" 
```
---------------


## sed基础
```bash
sed [参数][地址commands][inputfile] 
```
sed 工作流程一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（patternspace ）.
接着用sed 命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。然后读入下行
![2020-02-11-14-57-18](http://noback.upyun.com/2020-02-11-14-57-18.png)

### 参数:
```bash
--version 查看版本
--help 查看帮助
-n 不输出模式空间内容到屏幕，即不自动打印,如上工作流程中说到，sed在输出的时候会先经过模式空间，在模式空间中处理完当前行后才会输出，而-n选项则是控制问在在模式空间输出完以后，不再输出
-e 多点编辑，允许多个脚本指令被执行
-r 支持正则
-i 修改原文件内容
-i.bak 在将处理的结果写入文件之前备份一份
-f 支持使用脚本文件
```

### sed自定义匹配分隔符
sed 可以使用其他的参数来代替默认的 斜杠来作为分隔符
但是这里有个小插曲，如果是没有使用处理规则的话， 分隔符貌似不适用

```bash
sed -n "/cy/p" # 成功
sed -n ",cy,p" # 失败
sed -n "s,cy,nosea,p" # 成功
```


### sed在mac和linux上的不同使用
在mac上sed的文档
```
-i extension
    Edit files in-place, saving backups with the specified extension.  If a zero-length extension is given, no backup will be saved.  It is not recommended to give a zero-
    length extension when in-place editing files, as you risk corruption or partial content in situations where disk space is exhausted, etc.
```
在linux上sed的文档
```
-i[SUFFIX], --in-place[=SUFFIX]
    edit files in place (makes backup if SUFFIX supplied)
```

因此我们用sed操作一个文件的时候，我们需要给-i添加一个指定的空字符串
```
sed -i ""  's#`#"#g' ./code_file.sh
```

### Command匹配段
sed 提供了一些匹配的手法来匹配内容后传入到模式空间的内容匹配大致的规则如下
![2020-02-11-15-09-02](http://noback.upyun.com/2020-02-11-15-09-02.png)
sed一共分为5段内容，第一段是匹配的内容，用来确定当前段是否需要被传入模式空间中修改
```bash
不给地址 :对全文进行处理
单地址 :
    - #: 由数字指定第几行,但是数字指定的时候，不需要用分隔符隔开
    # 如 sed -n '1p' 第一行
    - /pattern/: 
    # 如sed -n '/root/p'
    - #,#  
    # 如 sed -n '1,2p' 第一行到第二行
    - /part1/,/part2/  
    # 如 sed -n '/root/,/docker/p' /etc/passwd
步进 ～
    - sed -n '1~2p' 只打印奇数行,起始为1 步进为2
    - sed -n '2~2p' 只打印偶数行 
    - # 单次步进  
    #,+# 如 sed -n '1,+2p' 第一行到第三行
```

### 处理规则

第二段是处理规则的内容，经过第一段筛选过后，会根据不同的处理规则进行转换

```bash
s 替换 
# s/pattern/pattern1/ 将pattern 替换成pattern1  只有在s替换的时候才可以把/ 换成其他的内容
# sed -n  '/bbb/s#bb#aa#p' yy.sh
d 删除 # sed '1d' xx.sh  sed '/bbb/d' xx.sh 删除匹配行，删除模式空间匹配到的行
i\ 从当前行前插入 # sed '1 i xx' xx.sh  sed '/bbb/i\xx' xx.sh 
a\ 在当前行后插入 
# sed '1 a xx' xx.sh 
# sed '/bbb/a\xx' xx.sh  
# sed '/bbb/axx' xx.sh 
c\ 替换匹配行
# sed '1 c xx' xx.sh
# sed '/bbb/c\xx' xx.sh
# sed '/bbb/cxx' xx.sh

y 对应替换，与s 不一样的是y需要对应字符数量
# sed '/bbb/y/BBB/' xx.sh
```
i a c三个内容需要注意，他们后面并没有第三段和第四段以及第五段内容,且只能加反斜杆表转义

### 处理方式
最后一个段是选择处理方式 
```bash
g 匹配所有行
p 打印当前模式空间内容，追加到默认输出之后
w 写入一个文件
# sed '/My/w' xx.sh 写入到xx.sh文件里面
r 读取一个文件
# sed '/My/r' xx.sh 读取xx.sh文件里面的My 
```
可以一起使用
### 使用案例
```bash
[root@hzj ~]# cat demo
aaa
bbbb
AABBCCDD
# 匹配/aaa/行
[root@hzj ~]# sed "/aaa/p" demo  #aaa进入模式空间，/p打印模式空间内容。出现两遍aaa
aaa
aaa
bbbb
AABBCCDD 
# 匹配aaa行并仅输出
[root@hzj ~]# sed -n "/aaa/p" demo  #-n关闭自动输出，/p打印模式空间内容，出现一遍aaa
aaa
[root@hzj ~]# sed -e "s/a/A/" -e "s/b/B/" demo  #-e多点编辑
Aaa
Bbbb
AABBCCDD
[root@hzj ~]# cat sedscript.txt
s/A/a/g
[root@hzj ~]# sed -f sedscript.txt demo  #-f使用文件处理
aaa
bbbb
aaBBCCDD
[root@hzj ~]# sed -n "p" demo  #关闭默认输出。p输出模式空间内容
aaa
bbbb
AABjBCCDD
[root@hzj ~]# sed "2s/b/B/g" demo  #替换第2行的b->B
aaa
BBBB
AABBCCDD
[root@hzj ~]# sed -n '1,3p' /etc/passwd # 输出第1,3行

[root@hzj ~]# sed -n '/^root/,/daemon/p' # 输出passwd文件的root～daemon行

[root@hzj ~]# sed -n '/x:[0:1]:/p' /etc/passwd #输出passwd文件中uid是0或1的行

# 删除第一行
sed '1d' /etc/passwd 
删除第一行和第三行
sed -e '1d' -e '3d' /etc/passwd
sed  '1d;3d' /etc/passwd
保留第一行 删除其他行
sed '1!d' 
在第二行后加123
sed "2a123" demo 
在第一行前加123
sed "1i123" demo
保存第三行的内容到demo3文件中
sed -n "3w/root/demo3" demo
读取demo3的内容到第1行后
sed "1r/root/demo3" demo  
先匹配后替换
sed -e '/^root/s/^1/2' /etc/passwd
匹配第一个并修改
sed 's/root/ROOT' /etc/passwd
匹配第二个并修改
sed 's/root/ROOT' /etc/passwd
匹配所有并修改
sed 's/root/ROOT/g' /etc/passwd
匹配修改成功后，打印变动行
sed -n 's/root/ROOT/pg' /etc/passwd
修改后另存为其他文件
sed -n 's/root/ROOT/pg;w passwd.md' /etc/passwd
sed -n 's/root/ROOT/pgw passwd.md' /etc/passwd
在文件的每行前面添加#注释
sed -n 's/^/#/p' /etc/passwd
删除文件的第1个字符
sed 's/^.//1' /etc/passwd
删除文件的第二个字符
sed 's/.//2' /etc/passwd
在每一行的上面添加hello 
sed 'i hello' /etc/passwd
在第一行的上面添加hello
sed '1a hello' /etc/passwd
将a~z替换成A～Z 
sed 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/' /etc/passwd
替换匹配行
sed -n '1c hello' /etc/passwd
输出1行
sed -n '1q' /etc/passwd
打印奇数行
sed -n 'p;n' /etc/passwd
打印偶数行
sed -n 'n;p' /etc/passwd
打印匹配字符串的下一行
sed -n '/^root/{n;p}' /etc/passwd 
# 打印包含pattern的行
sed 'root/p' /etc/passwd | less # 输出匹配行+全文
sed -n 'root/p' /etc/passwd # 输出匹配行
sed -n '/^root/p' /etc/passwd # 输出以root开头行
sed -n 'root/!p' /etc/passwd # 输出以root开头以外的行

# 删除pattern的行
sed '/^root/,/shutdown/d' /etc/passwd 
# 步进多次
sed -n '/^root/,~3p' /etc/passwd 
# 步进一次 
sed -n '/^root/,+3p' /etc/passwd  
# 匹配行的下一行
sed '/^root/{n;d}' /etc/passwd 

# 行插入  追加 修改
sed -r '/^root/i\>>>> test by geminis' /etc/passwd
sed -r '/^root/a\>>>> test by geminis' /etc/passwd
sed -r '/^root/c\>>>> test by geminis' /etc/passwd

# 删除空格
sed s/[[:space:]]//g file
# 删除空格 
sed /^$/d file

```
---------

## awk基础
awk是由Alfred Aho 、Peter Weinberger 和 Brian Kernighan这三个人创造的，awk由这个三个人的姓氏的首个字母组成。awk其实可以看成是一门独立的编程语言，他支持条件判断、数组、循环等功能。awk共有两个版本(gawk与nawk版本),在linux中我们最常使用的是gawk,同时awk、grep、sed被称为Linux中的三剑客

```bash
[root@CodeSheep ~]# ll /usr/bin/awk 
lrwxrwxrwx. 1 root root 4 Feb 15  2019 /usr/bin/awk -> gawk
```

### awk的工作流程
首先awk并不是列操作，而是行操作，与sed相同的是，他同样是一行一行执行处理程序，每一行进入awk中，首先会读取分隔符号
分隔符号可以由你自己定义，默认为空格

### awk中的分隔符

为了处理好每一行中的每一个字段，awk引入了分隔符的概念，分隔符有三种表现的形式，
- 一种是不输入任何不指定任何，则默认为空格
- 一种是自定义的分隔符 使用 -F 指定分隔符  比如我这里我们指定冒号为分隔符  比如 -F: 
- 另外一种是使用 内置变量指定  -v FS='#'
- 另外一种是在BEGIN中去定义 awk 'BEGIN{FS='#' }{print  $0}'

<font color='red'>输入分隔符</font>
既然默认的分隔符是空格,那么当然还有自定义的分隔符咯。我们使用-F选项来指定我们的分隔符
```bash
# 使用#分隔符
[root@CodeSheep ~]# cat demo3.text 
123#123#dsdshj#dlsj#
[root@CodeSheep ~]# awk -F# '{print $1}' demo3.text 
123
[root@CodeSheep ~]# cat demo3.text 
123#123#dsdshj#dlsj#
[root@CodeSheep ~]# awk -v FS='#' '{print $1}' demo3.text 
123
```
<font color='red'>输出分隔符</font>

- 默认为空格
- 指定输出分隔符  -v OFS=‘++++’
```bash
# 默认分隔符
[root@CodeSheep ~]# awk -F# '{print $1,$2}' demo3.text 
123 123
# 指定分隔符
[root@CodeSheep ~]# awk -v  FS='#' -v OFS='++++' '{print $1,$2}' demo3.text 
123++++123
```

### awk格式化输出
和c类似,不需要格式化的话，用print 需要格式化的话，用printf
<font color='red'>与print函数不同的是， printf不会在行尾自动换行。因此，如果要换行，就必须在控制串中提供转义字符\n。</font>
另外printf使用占位符来格式化输出

占位符如下
```bash
-c 字符
-s 字符串
-d 十进制整数
-ld 十进制长整数
-u 十进制无符号整数
-f 浮点数
```

修饰符如下
```bash
- 左对齐修饰符
\# 显示8 进制整数时在前面加个0
\# 显示16 进制整数时在前面加0x

+ 显示使用d 、e 、f 和g 转换的整数时，加上正负号+或-
0 用0而不是空白符来填充所显示的值
```

域
域是用来控制输出格式的，在占位符中定义，如下，域为15
```bash
echo "Linux" | awk '{printf "|%-15s}|",$1}'
# res: |Linux      |
```

### awk的内建变量
awk支持内建变量和自定义变量

#### 分割内建变量
- $0 当前行
- $1 第n个字段
- $NF 最后一个字段
- $(NF-1) 倒数第二个字段
- NF (number of field)当前行被分割字段总数
- NR (number of record记录)当前行行号
- FNR (file number of record)各文件分别计数的行号
- FS (filed Separator分割，分离) 字段分隔符
- OFS （output filed separated）输出字段分隔符
- RS (Row Separator) 换行分隔符 默认为空格
- ORS（output Row separated ) 输出换行分隔符

```bash
# 准备文件
[root@hzj-machine tmp]# cat xx
 111 222
 333 444
 555 666
[root@hzj-machine tmp]# cat yy
111 222
333 444
555 8888
# 输出当前行，因为是流处理，所以会输出所有的内容
[root@hzj-machine tmp]# awk '{print $0}' xx
 111 222
 333 444
 555 666
# 输出每一行的第n个字段
[root@hzj-machine tmp]# awk '{print $1,"----",$2}' xx
111 ---- 222
333 ---- 444
555 ---- 666

# 输出每一行的最后一个字段
[root@hzj-machine tmp]# awk '{print $2,"----",$NF}' xx
222 ---- 222
444 ---- 444
666 ---- 666

# 输出每一行被分割字段的总数
[root@hzj-machine tmp]# awk '{print $1,"----",$2,"sum field:"NF}' xx
111 ---- 222 sum field:2
333 ---- 444 sum field:2
555 ---- 666 sum field:2

# 输出当前行号 NR
[root@hzj-machine tmp]# awk '{print NR,$0}' xx
1  111 222
2  333 444
3  555 666
# 输出各文件分别计数的行号 FNR
[root@hzj-machine tmp]# awk '{print FNR,$0}' xx yy
1  111 222
2  333 444
3  555 666
1 111 222
2 333 444
3 555 8888

# 准备文件
[root@hzj-machine tmp]# cat xx
 111#222
 333#444
 555#666

# 指定切割符
[root@hzj-machine tmp]# awk  -v FS='#' '{print $1}' xx
 111
 333
 555
# 指定输出分隔符
[root@hzj-machine tmp]# awk  -v FS='#' -v OFS='----' '{print $1,$2}' xx
 111----222
 333----444
 555----666

 # 输出记录分隔符
[root@hzj-machine tmp]# awk  -v FS='#' -v OFS='----' '{print $1,$2,RS}' xx
 111----222  ----
 333----444  ----
 555----666 ----

# 输入换行符
[root@hzj-machine tmp]# awk -v RS='#' '{print $0}' xx
 111
222
 333
444
 555
666
# 输出换行符
[root@hzj-machine tmp]# awk -v RS='#' -v ORS='-----' '{print $0}' xx
 111-----222
 333-----444
 555-----666
```
#### 内建环境变量
- FILENAME 当前文件名
- ARGC 命令行参数的个数
- ARGIND 当前文件在ARGC中的位置

- ARGV 数组,保存的命令行所给定的各参数
```bash 
# ARGV,ARGC
[root@CodeSheep ~]# awk 'BEGIN{print ARGV[0],ARGV[1],ARGV[2],ARGC}' demo1.txt demo2.text 
awk demo1.txt demo2.text 3
  # 其中ARGV作为数组,第一个参数是awk
```
#### 关系运算符
```bash
<   小于      x < y
<=  小于等于  x <= y
==  等于      x==y
!=  不等于  x!=y
> = 大于等于 x>=y
>   大于  x>y
~  正则匹配  x ~ /正则/
!~ 正则不匹配  x !~ /正则/
```


#### 自定义变量
自定义变量的两种形式
- 一种是在action外面 使用选项指定变量，比如 -v name="hzj"
- 另外一种则是在action前的pattern中定义，比如 {name="hzj"; action->print name}
- 直接在程序中声明 name=hzj 
```bash
[root@CodeSheep ~]# awk -v name="hzj" 'BEGIN{print name}'
hzj
[root@CodeSheep ~]# awk 'BEGIN{name="hzj";print name}'
hzj
[root@CodeSheep ~]# name=hzj
[root@CodeSheep ~]# awk -v name=$name 'BEGIN{print name}'
hzj
# awk '{print $name}' xx 
# 这样写是错误的
```



### awk-options部分
- 指定变量
  -v name="xx"   -v外部定义变量

### awk-action部分
```bash 
awk [options] '{pattern + action}' {filenames}  
```
其中的action 我们最常用的就是print以及prinf,对于action来说，每次经过一行，都会当前行执行一边action
比如
```bash
awk '{print $0,NR}' /etc/passwd 
```
**抛开Pattern 我们先来使用{action}**
```bash
# 输出文本内容
[root@CodeSheep ~]# df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/vda1       41147472 6848500  32395480  18% /
devtmpfs          930656       0    930656   0% /dev
tmpfs             941116       0    941116   0% /dev/shm
tmpfs             941116     452    940664   1% /run
tmpfs             941116       0    941116   0% /sys/fs/cgroup
tmpfs             188224       0    188224   0% /run/user/0
[root@CodeSheep ~]# df | awk '{print $1}'
Filesystem
/dev/vda1
devtmpfs
tmpfs
tmpfs
tmpfs
tmpfs
```
awk是逐行处理的,也就是说当awk处理一个文本的时候，他会一行一行的处理内容。其中awk默认以 <font style='color:red'>换行符</font>  为标记识别每一行。另外对于每一行的处理中 awk会按照用户指定的 <font style='color:red'>分隔符</font> 去分隔当前行，如果没有指定，则默认使用空格作为分隔符，当出现多个空格的时候,awk会自动将连续的空格理解成为一个分隔符。
其中 我们将被awk处理后的每一行都使用了特定的变量标记

```bash
[root@CodeSheep ~]# df | awk '{print "$1="$1 , "testfield:""this print test"}'
$1=Filesystem testfield:this print test
$1=/dev/vda1 testfield:this print test
$1=devtmpfs testfield:this print test
$1=tmpfs testfield:this print test
$1=tmpfs testfield:this print test
$1=tmpfs testfield:this print test
$1=tmpfs testfield:this print test
#看上面的案例我们可以发现,当$1被双引号包裹的时候，他就是一个字符串,不再是变量
```

### awk中的Pattern部分
其实action的主要操作就是print输出,简单来用就是输出内容，更复杂的就是对输出的内容进行格式化的操作。而Pattern所存在的两种模式,愈加加强了awk的能力。
为了更好的格式化,awk中也带有逻辑参数，其中包括开始BEGIN和结尾END,他们之间用{}分隔,比如
```bash
awk -v test="test" 'BEGIN{print "1",NR}{print test}END{print "input end" }' /etc/passwd
```
你会发现他输出了很多个test，首先begin进入，然后输出1和行号0 紧接着不断的进入行输出test 在最后一行输入时 执行input end

**特殊模式下的BEGIN与END**
- BEGIN 模式指定了处理文本之前需要执行的操作
- END 模式指定了处理完所有行之后所需要执行的操作

```bash
[root@CodeSheep ~]# df | awk 'BEGIN{print "$1","$2"}'
$1 $2
[root@CodeSheep ~]# df | awk 'BEGIN{print "$1","$2"} {print $1,$2}'
$1 $2
Filesystem 1K-blocks
/dev/vda1 41147472
devtmpfs 930656
tmpfs 941116
tmpfs 941116
tmpfs 941116
tmpfs 188224
[root@CodeSheep ~]# df | awk 'BEGIN{print "$1","$2"} {print $1,$2} END{print "end for file"}'
$1 $2
Filesystem 1K-blocks
/dev/vda1 41147472
devtmpfs 930656
tmpfs 941116
tmpfs 941116
tmpfs 941116
tmpfs 188224
end for file
```
在BEGIN的模式下，首先awk会执行BEGIN中的action 之后才做去执行其他的action,同理,在执行完所有的action之后,awk回去执行END模式下面的action。<font style='color:red'>需要注意的是两个特殊的模式BEGIN以及END都需要大写</font>

于是 awk的格式化能力表露无疑了,提取字段再加上BEGIN与END的两种特殊模式，一张完整的表不就出来了嘛


### awk中的正则模式
既然上面的关系运算符中出现了正则，那我们就来讲一讲正则模式
需要注意的是 当使用{x,y}这种次数匹配的正则表达式时，需要配合--posix选项或者--re-interval选项
```bash
# 匹配/etc/passwd 中以 tss开头的行
[root@CodeSheep ~]# cat /etc/passwd |  awk '/tss/{print $0}'
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
# 匹配/etc/passwd 中以 /bin/bash结尾的行
[root@CodeSheep ~]# cat /etc/passwd |  awk '/\/bin\/bash$/{print $0}'
root:x:0:0:root:/root:/bin/bash
# 匹配/etc/passwd 中以/bin/bash结尾的行 到以 tss开头的行
[root@CodeSheep ~]# cat /etc/passwd |  awk '/\/bin\/bash$/,/^tss/{print $0}'
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
ntp:x:38:38::/etc/ntp:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
nscd:x:28:28:NSCD Daemon:/:/sbin/nologin
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/false
dockerroot:x:997:994:Docker User:/var/lib/docker:/sbin/nologin
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
# 匹配行号大于2 且小于6的行
[root@CodeSheep ~]# cat /etc/passwd |  awk 'NR>2 && NR<=6{print $0}'
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
```

### awk中的流程控制
在awk中,同样可以使用if for while等在编程语法中出现的逻辑判断
- if 语句
- while 语句
- switch 语句
- getline 跳转语句
getline跳转语句能够更好的控制流程，他会得到当前行的下一行，比如我们输出奇数行
```bash
seq 10 | awk '{print $0;getline}'
1
3
5
7
9
```
此外，getline也可以用来执行一个UNIX命令，并得到它的输出。比如输出系统时间
```bash 
awk 'BEGIN {"date" | getline; close("date"); print $0}'
```
另外，我们还可以使用getline来判断是否当前行为最后一行，这样做的目的比如我们在格式化一个表格成为json文件的时候，最后一个不需要逗号，我们可以那这个作为判断依据
```bash
# target_file type like this 
# CUN-FJ-XMN-S01 ansible_ssh_host=36.248.208.226 uplink=T2/0/49
# CUN-SD-DEZ-S01 ansible_ssh_host=61.179.240.158 uplink=T1/0/49
# CTN-SN-XIY-S01 ansible_ssh_host=1.81.5.161 uplink=B100
cat $target_file | grep "ansible_ssh_host"  | awk  'BEGIN{
    printf("\{\n \"instances\": \[\n");
}{
       split($2,host,"=");hostIp=host[2];
       split($3,uplinkArray,"=");uplink=uplinkArray[2];

        # 这里getline用来识别是否跳转到了最后一行 1表示还有 0表示没有下一行，也就是最后一行了
       if ( getline == 1){
           printf("{\"name\":\"%s\",\"host\":\"%s\",\"uplink\":\"%s\"},\n",$1,hostIp,uplink);
        }else{
            printf("{\"name\":\"%s\",\"host\":\"%s\",\"uplink\":\"%s\"}\n",$1,hostIp,uplink);
        }
        

    }END{
        print("\]\}")
    }'
```

- close 关闭语句
```bash
echo "21 2
52
23" | awk '{
first[NR]=$1
second[NR]=$2
}END{
print "======打印第1列并排序：===========" > "testAwkPipe.txt"
close("testAwkPipe.txt")
for(i in first)
  print first[i] |"sort -n >> testAwkPipe.txt"
close("sort -n >> testAwkPipe.txt")

print "======打印第2列并排序：===========" >> "testAwkPipe.txt"
close("testAwkPipe.txt")
for(i in second)
  print second[i] |"sort -n >> testAwkPipe.txt"
}
close("sort -n >> testAwkPipe.txt")
'
```



```bash
# 输出df第一行
[root@CodeSheep ~]# df | awk 'NR==1{print $0}'
Filesystem     1K-blocks    Used Available Use% Mounted on
[root@CodeSheep ~]# df | awk '{if(NR==1){print $0}}'
Filesystem     1K-blocks    Used Available Use% Mounted on
```
if(){语句1;语句2;}
if(){语句1;语句2;}else{语句3;语句4;}
if(){语句1;语句2;}else if(){语句3;语句4;}else{语句5;语句6;}


for(初始化;表达式;更新){语句}
for(变量 in 数组){语句}
while(表达式){语句}
do{语句}while(条件)

### awk中的内置函数
awk中有很多的内置函数，这些函数在其他的语言中也会有部分的实现
- split 切割
awk的内建函数split允许你把一个字符串分隔为单词并存储在数组中。你可以自己定义域分隔符或者使用现在FS(域分隔符)的值。
```bash 
# 使用方式
awk '{   split (string, array, field separator)   }'
awk -v FS="#" '{   split (string, array) }'  # 如果这里像上面那样去处理的话，split会使用awk中定义的FS，默认情况下awk的FS为空格
# awk指定多个分隔符
awk -v FS="[|= :]" '{split (string,array)}'
# string 表示你要切割的字符串 或者行$0 $1等等
# array 表示切割后存放的数组名  提取 first=array[0]
# fs 表示切割符
```
例子来一个
```bash
# 创建测试
root:/tmp# cat xx
12:234234:54354:654645
# 切割
root:/tmp# awk '{split($0,list,":");print list[2]}{print list[1]}' xx
234234
12
```
- substr 截取字符串
返回从起始位置起，指定长度之子字符串；若未指定长度，则返回从起始位置到字符串末尾的子字符串。
```bash 
substr(string,start) 返回字符串s中从p开始的后缀部分
substr(string,start,step) 返回字符串s中从p开始长度为n的后缀部分

root:/tmp# echo "abc" | awk '{print substr($0,2,2)}'
bc
```
- length 字符串长度
```bash 
root:/tmp# echo "abc" | awk '{print length}'
3
root:/tmp# awk 'BEGIN{a[1]=2}{print length(a) }'
1
```
- gsub函数则使得在所有正则表达式被匹配的时候都发生替换。gsub(regular expression, subsitution string, targe string);
相比于sub来说 gsub则是替换所有 第一个是正则表达式，第二个是需要被替换的内容，第三个是被操作对象
```bash
>> echo "abc cde abr abc abc" | awk '$0 ~ /abc/ {sub("abc","def",$0); print $0}'
def cde abr abc abc
>> echo "abc cde abr abc abc" | awk '$0 ~ /abc/ {gsub("abc","def",$0); print $0}'
def cde abr def def

# abc是正则表达式 
# def 表示替换的内容
# $0 是操作的内容
```


### awk中的数组
使用awk awk中的数组功能与一些语言的数组功能类似
比如
```bash 
[root@hzj-machine ~]# awk 'BEGIN{a["xx"]=1;a["xx"]=1}{for ( i in a){print a[i],i}}' xx
1 xx
# 输出 元素 a[i]
# 输出下标 i
```
在awk中，默认情况下，awk的下标是从1开始的，但是awk有一定的自由性，他允许数组下标的任意性
<font color='red'>比如我们可以设置awk的下标为一个字符串，如上codeBlock,也允许下标为一个空字符串，如下</font>  
![2020-03-10-10-56-23](http://noback.upyun.com/2020-03-10-10-56-23.png)

并且在想要取一个不存在的元素时候，awk会自动创建这个元素，并且默认取值为空字符串
![2020-03-10-10-59-46](http://noback.upyun.com/2020-03-10-10-59-46.png)

判断下标是否存在,以及if 取反
```bash 
[root@hzj-machine ~]# awk 'BEGIN{a["    "]=1;a[2]=3}END{ if ( 2 in a ){ print "3存在"}else{print "下标2不存在"}}' xx
3存在
[root@hzj-machine ~]# awk 'BEGIN{a["    "]=1;a[2]=3}END{ if(! ( 2 in a )){ print "3存在"}else{print "下标2不存在"}}' xx
下标2不存在
```
删除数组元素,以及删除数组
```bash 
awk 'BEGIN{a["    "]=1;a[2]=3}{delete a[2];delete a}}' xx
```

### awk中的数组实例
记录一个文本中同一ip出现的次数
```bash 
# 构建文本
[root@CMN-SC-CTU-036 getss]# cat h.txt
117.174.74.72 223.85.20.154:80  45.703 51 0  12.7  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
192.168.246.35 192.168.246.36:60244  1.791 54 0  349.3  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
192.168.246.35 192.168.246.36:60244  10.255 53 0  59.9  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
223.114.234.244 223.85.20.154:80  136.981 53 0  4.3  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
117.188.84.191 223.85.20.154:80  39.835 494 0  141.7  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
# 添加元素和下标到数组中
[root@CMN-SC-CTU-036 getss]# awk '{list[$2]++;}' h.txt
# 取数组的下标
[root@CMN-SC-CTU-036 getss]# awk '{list[$2]++;}END{for ( i in list) { print i}}' h.txt
223.85.20.154:80
192.168.246.36:6024
# 取数组下标s上的元素总共出现了几次
[root@CMN-SC-CTU-036 getss]# awk '{list[$2]++;}END{for ( i in list) { print i,list[i]}}' h.txt
223.85.20.154:80 3
192.168.246.36:60244 2
```
1. 首先awk同往常一样读取文本的第一行，遇到$2中的ip地址，因为没有这个下标，记录这个ip地址为下标，并且因为是头一次出现，awk的数组会为他创建一个空字符串，并且赋值为1  也就是说 list['223.85.20.154:80'] = 1

2. 第二次读取到$2的时候，同样没有出现过，操作步骤同上  list['192.168.246.36:60244'] = 1
3. 第三次遇到下标一样的内容，于是list['192.168.246.36:60244'] =2
4. 4到5次循环的时候，分别是  list['223.85.20.154:80'] = 2  list['223.85.20.154:80'] = 3


记录一个文本中同一IP下 其速度值的平均值

记录总值
```bash 
[root@CMN-SC-CTU-036 getss]# cat h.txt
117.174.74.72 223.85.20.154:80  45.703 51 0  12.7  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
192.168.246.35 192.168.246.36:60244  1.791 54 0  349.3  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
192.168.246.35 192.168.246.36:60244  10.255 53 0  59.9  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
223.114.234.244 223.85.20.154:80  136.981 53 0  4.3  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
117.188.84.191 223.85.20.154:80  39.835 494 0  141.7  ESTAB bbr network-attack 09/Mar/2020:15:41:34 CMN-SC-CTU-036
[root@CMN-SC-CTU-036 getss]# awk '{ a[$2]++;send[$2]+=$6}END{print send[$2],$2}' h.txt
158.7 223.85.20.154:80
```
输出数组中的长度
```bash 
[root@CMN-SC-CTU-036 getss]# awk '{ a[$2]++;send[$2]+=$6}END{print send[$2]/length(send),$2,length(send)}' h.txt
79.35 223.85.20.154:80 2

# 或者通过获取下标的形式获得长度
[root@CMN-SC-CTU-036 getss]# awk '{ a[$2]++;send[$2]+=$6}END{for ( i in send){print i,send[i],send[i],a[i]}}' h.txt
223.85.20.154:80 158.7 158.7 3
192.168.246.36:60244 409.2 409.2 2
```


### awk中调用shell 命令 system()
示例:
```bash
# 根据名字匹配pid 并关闭
lsof -i :80 | awk  '{ if ($1 ~ /uvicorn/ ||  $1 ~ /python/) { resPid=$2 ;cmd="kill -9  "resPid;system(cmd)} }'
```
在awk中通过system()来调用子shell处理命令，上面的命令就是通过匹配名字来kill pid

<font color='red'>踩坑</font>
这里需要注意一些东西，首先只要你是在awk 中声明的变量，引用的时候是不用加`$`的，比如上面的pid值
另外使用system()的时候 内部只能识别字符串。也就是说不能识别变量，
如果要识别变量，只能现在外部拼接一个变量然后传进去

```bash
# 方法1
system(" ")
# 方法2
cmd="kill -9"resPid
system(cmd)
```

## 错误
mac下sed的问题
如果需要在文本第一行前插入一行文字，应该用什么sed命令呢？

sed -e '1i hello'​ file.txt 应该就够了，但是在mac os x下却不行，系统会提醒你 ：
command i expects \ followed by text  
似乎是因为我缺少一个\的原因。如果你真的按提示写成 
sed -e '1i \hello' file.txt​,仍然不行，系统依旧会提示：
": extra characters after \ at the end of i command​
那么到底该怎么做呢 ，你应该在输完\后，按回车，此时系统会认为命令没有结束，光标移动到下一行，你接着输入后面的内容，居然就可以了。命令大概长成下面这个样。
```bash
sed -e '1i\
> hello' test.txt 
```



