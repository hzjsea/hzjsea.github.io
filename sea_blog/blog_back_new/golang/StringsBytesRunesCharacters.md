---
title: 'Strings, bytes, runes and characters in Go'
categories:
  - golang
tags:
  - golang
abbrlink: d9016abb
date: 2020-09-07 14:27:31
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Strings, bytes, runes and characters in Go](#strings-bytes-runes-and-characters-in-go)
  - [ASCII 码](#ascii-码)
  - [非ASCII编码](#非ascii编码)
  - [Unicode](#unicode)
  - [UTF-8](#utf-8)
  - [编码转换](#编码转换)
  - [格式化输出规定符](#格式化输出规定符)
  - [golang中的字符串字节符文和字符](#golang中的字符串字节符文和字符)
  - [golang各个字符类数据类型区别](#golang各个字符类数据类型区别)
  - [参考文章如下](#参考文章如下)

<!-- /code_chunk_output -->
<!-- more -->

# Strings, bytes, runes and characters in Go
Stirngs,bytes,runes,characters四个数据类型在golang中有很大的区别，官方有一篇文章专门来讲这四个数据类型在go中的区别
https://blog.golang.org/strings
https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/

::: tip 
什么是 位(bit) 字节(byte) 字(word)
:::

- 位(bit)   音译为比特  表示二进制位，位是计算机中最小的存储单位 11010100是一个8位二进制数。一个二进制位只可以表示0和1两种状态（2^1）；两个二进制位可以表示00、01、10、11四种（2^2）状态；三位二进制数可表示八种状态（2^3）……
- 字节（byte） 字节来自英文Byte，音译为“拜特”，习惯上用大写的“B”表示。 字节是计算机中数据处理的基本单位。计算机中以字节为单位存储和解释信息，规定一个字节由八个二进制位构成，即1个字节等于8个比特（1Byte=8bit）。八位二进制数最小为00000000，最大为11111111；通常1个字节可以存入一个ASCII码，2个字节可以存放一个汉字国标码。
- 字 计算机进行数据处理时，一次存取、加工和传送的数据长度称为字（word）。一个字通常由一个或多个（一般是字节的整数位）字节构成。例如286微机的字由2个字节组成，它的字长为16；486微机的字由4个字节组成，它的字长为32位机。 计算机的字长决定了其CPU一次操作处理实际位数的多少，由此可见计算机的字长越大，其性能越优越。


换算单位  `1word` = `n*1byte` = `n*8bit` 


::: tip 
不同长度的整型出现的原因
:::
![2020-10-22-10-54-16](http://noback.upyun.com/2020-10-22-10-54-16.png)
主要还是因为历史原因，历史上，不是所有计算机都是一个字节 8 个 bit 的。只是由于一系列优势 (比如刚好能容纳下 ASCII 码 + 1 位校验位)，因为这一优势，最后形成了现在常见的架构，也就是1个byte 表示8个bite的架构
于是后面的int8 int16 int32都是在这之上扩展的，这样的做的主要原因是，可以节省内存。如果想Python这样不考虑省内存， 32/64位之内都用int ,超过就用long。如果所有语言都选择使用了long 这样就比较占内存的了。



## ASCII 码
位(bit)是计算机最小存储单位， 8个bit组成1个字节，也就是由0和1组成256种不同的状态。每一个状态对应一个符号
上个世界60年代，美国指定了一套字符编码，对英文字符与二进制位之间的关系，做了统一的规定，被叫做ASCII码
ASCII 码一共规定了128个字符的编码，比如空格SPACE是32（二进制00100000），大写的字母A是65（二进制01000001）。这128个符号（包括32个不能打印出来的控制符号），只占用了一个字节的后面7位，最前面的一位统一规定为0。
ASCII码查询 https://tool.oschina.net/commons?type=4

## 非ASCII编码
英语有128个符号编码就可以解决了，但是用来表示其他语言，128个符号是不够的。比如，在法语中，字母上方有注音符号，它就无法用 ASCII 码表示。于是，一些欧洲国家就决定，利用字节中闲置的最高位编入新的符号。比如，法语中的é的编码为130（二进制10000010）。这样一来，这些欧洲国家使用的编码体系，可以表示最多256个符号。但是，这里又出现了新的问题。不同的国家有不同的字母，因此，哪怕它们都使用256个符号的编码方式，代表的字母却不一样。比如，130在法语编码中代表了é，在希伯来语编码中却代表了字母Gimel (ג)，在俄语编码中又会代表另一个符号。但是不管怎样，所有这些编码方式中，0--127表示的符号是一样的，不一样的只是128--255的这一段。
到了中国，中华文化博大精深，文字也很多，256个字符完全无法表达所有的汉字， 于是简体中文采用了GB2312 ，使用两个字节来表示一个汉字 ，也就是256*256= 65536g个字符

## Unicode
因为非ASCII编码的出现，破幻了字符之间的统一，同一个二进制数字可以被解释成为不同的符号，不同国家在交换文件的时候，首先要解决的就是编码问题。于是后面就有了Unicode，他是一个很大的集合，可以容纳100W+个符号，每个符号的编码都不一一样。+0639表示阿拉伯字母Ain，U+0041表示英语的大写字母A，U+4E25表示汉字严

::: tip 
Unicode的问题 
:::
Unicode的出现解决了世界上符号不统一的问题，但同时也带了其他的问题，他只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储。
比如，汉字严的 Unicode 是十六进制数4E25，转换成二进制数足足有15位（100111000100101），也就是说，这个符号的表示至少需要2个字节。表示其他更大的符号，可能需要3个字节或者4个字节，甚至更多。
它们造成的结果是 
- 出现了 Unicode 的多种存储方式，也就是说有许多种不同的二进制格式，可以用来表示 Unicode
- Unicode 在很长一段时间内无法推广，直到互联网的出现。


## UTF-8
UTF-8 展开的意思就是 8-bit Unicode Transformation Format，它只是Unicode展现的一种形式，另外还有UTF-16(字符用两个字节或四个字节表示) 等等都没有能够普及
UTF-8 最大的一个特点，就是它是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度。

- 对于单字节的符号，字节的第一位设为0，后面7位为这个符号的 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的。
- 对于n字节的符号（n > 1），第一个字节的前n位都设为1，第n + 1位设为0，后面字节的前两位一律设为10。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码。
- 
![2020-10-22-14-10-24](http://noback.upyun.com/2020-10-22-14-10-24.png)

于是我们可以用每组开头的内容来判断是多少字节的，如果第一组开头是0 则是单字符，  如果第一组开头是1 则连续有多少个1，当前字符就占用多少个字节。  另外对于单字符来说，前面第一个是0 后面的7个所组成的内容和ASCII相同

举例
严的 Unicode 是4E25（100111000100101），根据上表，可以发现4E25处在第三行的范围内（0000 0800 - 0000 FFFF），因此严的 UTF-8 编码需要三个字节，即格式是1110xxxx 10xxxxxx 10xxxxxx。然后，从严的最后一个二进制位开始，依次从后向前填入格式中的x，多出的位补0。这样就得到了，严的 UTF-8 编码是11100100 10111000 10100101，转换成十六进制就是E4B8A5


::: tip 
顺便提一句，中文windows下面，默认使用的是GB2312编码，Linux下面的是UTF-8编码 
:::


## 编码转换
编码查看
`file filename`
或者在vim中查看编码
```bash
vi xx.txt
set fileencoding
# 设置编码
set fileencoding=utf-8
```
在`.vimrc`中设置编码格式，这样vim在打开的时候回自动判断编码，保证不会出现乱码
```bash
" 设置新文件的编码为iUTF-8"
set encoding=utf-8
" 自动判断编码时，依次尝试一下的编码"
set fileencodings=usc-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
```
跨平台之间难免会存在一些编码问题，在linux下可以使用iconv转换编码
![2020-10-22-14-31-16](http://noback.upyun.com/2020-10-22-14-31-16.png)
批量处理
```bash
#!/bin/bash
# 功能：将GB2312文件 转换成 UTF-8【解决Windows文件复制到Linux之后乱码问题】
#read -p "Input Path:" SPATH
SPATH="."
#echo $SPATH
POSTFIX="m"
param1="$1"
if [ "$param1" == "win" ];then
   sys1="Linux"
   sys2="Windows"
   format1="UTF-8"
   format2="GB2312"
elif [ "$param1" == "linux" ];then
   sys1="Windows"
   sys2="Linux"
   format1="GB2312"
   format2="UTF-8"
else
   echo "************** 功能 ************"
   echo "  解决matlab脚本文件在Windows和Linux中移动时出现的乱码问题！"
   echo "  将该脚本复制到程序文件夹中，运行该脚本，它会对当前文件夹及子文件夹中的所有*.m文件进行格式转换，解决乱码问题。"
   echo "  转换到 Linux 的命令: $0 linux"
   echo "  转换到 Window的命令: $0 win"
   exit
fi

echo "********************************"
echo "  格式转换中......"
echo "  从"$sys1"("$format1") 转换到 "$sys2"("$format2")"
echo "********************************"


FILELIST(){
filelist=`ls $SPATH `
for filename in $filelist; do
    if [ -f $filename ];then
        #echo File:$filename
        #echo "${filename#*.}"
        EXTENSION="${filename#*.}"
        #echo $EXTENSION
        if [ "$EXTENSION" == "$POSTFIX" ];then
           #echo "${filename%%.*}"
           echo Processing: $filename
           iconv -f $format1 -t $format2 $filename -o $filename
           #iconv -f GB2312 -t UTF-8 $filename -o $filename
        fi

    elif [ -d $filename ];then
        cd $filename
        SPATH=`pwd`
        #echo $SPATH
        FILELIST
        cd ..
    else
        echo "$SPATH/$filename is not a common file."
    fi
done
}
cd $SPATH
FILELIST
echo "======== Convert Done. ========"
```

## 格式化输出规定符
格式化输出固定符号，基本每个语言都是相同的
%d 十进制有符号整数
%u 十进制无符号整数
%f 浮点数
%s 字符串
%c 单个字符
%p 指针的值
%e 指数形式的浮点数
%x, %X 无符号以十六进制表示的整数
%0 无符号以八进制表示的整数
/n 换行
/f 清屏并换页
/r 回车
/t Tab符
## golang中的字符串字节符文和字符
https://blog.golang.org/strings



## golang各个字符类数据类型区别
首先来看单字节的 byte 和unit8，他们是一样都，都是8个bite位 ，表示1个byte。单字节的字符遵守ASCII码表的内容，于是内容如下
```go
var a byte = 65
// 8进制写法
var c byte = '\101'
// 16进制写法
var d byte = '\x41'
var b uint8 = 66
fmt.Printf("65 is %c, 66 is %c\n", a, b) // 65 is A , 66 is B
fmt.Printf("%c,%c\n", c, d) // A,A
```
接下来是4个byte的内容，也就是uint32的内容，他们表示用来表示Unicode字符
如何区分Unicode与ASCII 就看第一组的前缀是否为0

```go
var xx rune = 'A'
var yy rune = 'B'
var qq = 'C'
fmt.Printf("x占用字节数为%d", unsafe.Sizeof(x))                            // 4
fmt.Printf("xx,yy占用字节数为%d,%d", unsafe.Sizeof(xx), unsafe.Sizeof(yy)) // 4,4
fmt.Printf("x占用字节数为%d", unsafe.Sizeof(qq))                           // 4
```
默认情况下字符都是以rune的形式出现的 ，也就是表示为 uint32 
使用rune的好处就是可以保存更多的字符类型，包括中文，内容如下
```go
var qqxx rune = '中'
fmt.Printf("qqxx size is %d\n", unsafe.Sizeof(qqxx)) // 4
//var qqxxx byte = '中' // 这里报错 超出字节大小   constant 20013 overflows byte
//fmt.Println(unsafe.Sizeof(qqxxx))


var xx interface{} = "hellowrold"  // is string
var xxx interface{} = 'x' // is rune
switch xxx.(type) {
case string:
	fmt.Println("is string")
case rune:
	fmt.Println("is rune")
case byte:
	fmt.Println("is byte")
default:
	fmt.Println("dont know")
}


```

接下来是字符串，上面讲的都是字符，也就是用单引号包裹起来的，但是字符串则是用双引号包裹起来的。字符串的字节数为16
```go
fmt.Println("-------")
var zz string = "hello,中国"
var zzz string = "中"
fmt.Println(len(zzz)) // 3
fmt.Println(len(zz)) // 12
fmt.Println(unsafe.Sizeof(zz)) // 16
fmt.Println(unsafe.Sizeof(zzz)) // 16
var zzbyte []byte = []byte(zz)  
var zzzbyte []byte = []byte(zzz)
fmt.Println(zzbyte) // [104 101 108 108 111 44 228 184 173 229 155 189] 12
fmt.Println(zzzbyte) // [228 184 173] 3
```
golang中`len()`计算的是字符串的字节的长度

为什么有了uint8 和 uint32 还要出现byte和rune 这样比较好识别？
golang字符串可以表示为[]byte 和[]rune,因此很多方法也都类似。
比如

```go
// 将 s 中的所有字符修改为大写格式返回
var xx = "helloworld";
xx = strings.ToUpper(xx)
fmt.Println(xx) // HELLOWORLD
var ll []byte = []byte{'h','e','l','l','o','w','o','r','l','d'}
ll = bytes.ToUpper(ll)
fmt.Println(string(ll)) // HELLOWORLD
```

Fprintf使用
将内容写入到一个实现了Writer接口的变量当中 比如os.stadin
```go
fmt.Fprint(os.Stdin,"helloworld");
fmt.Println(os.Stdin) // helloworld
```
## 参考文章如下
https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU1NzU1MTM2NA==&action=getalbum&album_id=1337200768771866624&scene=173&from_msgid=2247483669&from_itemidx=1&count=3#wechat_redirect
https://mp.weixin.qq.com/s?__biz=MzU1NzU1MTM2NA==&mid=2247483673&idx=1&sn=09cc98b3f0b0e89e28c40a5f096c0d73&chksm=fc355b72cb42d26426bd4ae986ea46284b89d3bdf78465686a799684ddfc5f48f128382cc1ae&scene=21#wechat_redirect