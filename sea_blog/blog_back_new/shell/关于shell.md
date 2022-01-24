---
title: 关于shell
date: 2020-11-16 10:03:48
categories: [shell]  
tags: [shell]
---


<!-- more -->

# 关于shell
如果要在linux系统中愉快的执行脚本的话，最爽的还是shell脚本
shell脚本主要用来执行一些文本的批处理，自动化任务编写等等。

> shell可以看成一门独立的编程语言，但是和其他的编程语言不一样，shell的语法上有很多的不同，包括符号上的使用
> 这里做一下总结，不记所有，就记常用的


shell错误输出重定向
```bash
 command > /dev/null 2>&1
```

shell初始化模板
```bash
set -e 
#!/bin/bash
init() {
    os=$(uname -s)
    if [ $os=="Darwin" ]; then
        export frontcolor="\033[34m"
        export backcolor="\033[33m"
    elif [ $os== "Linux" ]; then
        export frontcolor="\e[34m"
        export backcolor="\e[33m"
    fi
}

_usage() {
echo -e "
${frontcolor}
**************************************
*       thank you use                *
*       Author: hzjsea               *
*       Date: you guess              *
**************************************

Use: xxxx
shortDesc: 构建单机redis
LongDesc: 构建单机redis

Command
    -n   add agrs
    -d   add args
    -z   add args
    -s   true or false
    -q   true or flase

Global Command
    -v
Example:
    ./xx.sh -n helloworld
${backcolor}
"
}

_leave() {
cat <<EOF
+-------------------------------------------------+
|  Thank you for usesing!                         |
+-------------------------------------------------+
EOF
}


_check() {
    echo "检查选项"
}

_stop() {
    echo "终止"
}

_status() {
    echo "获取状态"
}

down() {
    echo "关闭"
}

main() {

    init
    action=${1:-"default"}

    case ${action} in
    install | uninstall | update | start | stop | restart)
        _${action}
        ;;
    status)
        _${action}
        ;;
    script)
        check_script_update
        ;;
    uid)
        _${action}
        ;;
    default)
        _usage
        _leave
        ;;
    *)
        _usage
        _leave
        ;;
    esac

}

main $@

```


## 特殊值
- `$$` Shell本身的PID（ProcessID）
- `$!` Shell最后运行的后台Process的PID 
- `$?` 最后运行的命令的结束代码（返回值）,默认情况下 程序成功时，返回的是0，程序失败的时候返回的是1或者1+
- `$*` 所有参数列表
- `$@` 所有参数列表
- `$#` 添加到Shell的参数个数 
- `$0` shell本身的文件名
- `$1~$n` 添加到shell的各个参数值


`$* 与 $@`的区别就是 
当命令行为test.sh 1 2 3
`"$*“表示"1 2 3”`
`"$@“表示"1” “2” “3”`

## 比较判断

![2020-11-16-11-57-46](http://noback.upyun.com/2020-11-16-11-57-46.png)
![2020-11-16-11-58-02](http://noback.upyun.com/2020-11-16-11-58-02.png)
![2020-11-16-11-58-21](http://noback.upyun.com/2020-11-16-11-58-21.png)

> 先说一下 test 与 []

这两个是用来比较判断的，常用的是 `[]` 其实他们两个是一样的 无非是写法上的不同，一个是 `test expression` 而另一个是`[ expression ]` 注意这里必须要前后必须要有空格隔开，不然表达式无法被识别,当使用test判断成功时候，退出状态为 0，否则为非 0 值。

在`test`和`[]` 中 进行字符串比较时，使用`==和!=`
在`test`和`[]` 中 进行数值比较时，使用`-gt -eq`等等

```bash
#!/bin/bash
a=1
b=2

if [ $a -eq $b ];then
	echo "true"
else
	echo "false"
fi

a="hello"
b="world"
if [ $a == $b ];then
	echo "true"
else
	echo "false"
fi

```


> 单个括号 () 
> 命令的执行

正常情况下 `date` 与 `$(date)` 的效果是一样的 单个括号就是执行命令的意思，`$()`就是取命令的执行后的结果，以后建议使用后者


> 双小括号(())
> 用于计算

使用的情况如下`((expression))`
作用 
```bash
#!/bin/bash
#重新定义变量
a=5
((a++))
echo $a # 输出6

#for循环
for ((i = 0; i < 5; i++)); do
  echo 当前序号: $i
done

#条件判断
if ((1 + 2)); then
  echo 真
fi

if !((1 - 1)); then
  echo 假
fi

#算数运算
echo $((1 + 2 * 3)) # 先运算，后用$取值输出7
```

> [[]]
> 模式匹配

```bash
if [[ hello == hell? ]];then
  echo 'hello == hell?  true'
fi
if [[ 123 =~ ^[0-9]+ ]];then
  echo '123 match ^[0-9]+'
fi
```

## 变量

> 变量基础

shell中的变量声明不允许在等号附近出现空格



> 说一下普通变量引用与`${}`变量引用

正常情况下变量引用`$str`与`${str}` 是一样的，但是后者可以用更高级的匹配模式，以后建议常使用后者
语法
```bash
${变量名#匹配规则} 从头开始匹配并且删除，非贪婪
${变量名##匹配规则} 从头开始匹配并且删除 ,贪婪
${变量名%匹配规则} 从尾巴开始匹配并且删除，非贪婪
${变量名%%匹配规则} 从尾巴开始匹配并且删除，贪婪
${变量名/旧字符/新字符} 替换文字。只替换一次
${变量名//旧字符/新字符} 替换文字，全部替换
${file:start_index:step} 截取字符串
${#string} 返回字符串长度
```

> 说一下变量的几种形式

shell中变量分为环境变量,全局变量以及局部变量
默认情况下申明的都是全局变量

环境变量对子变量是有效果的,其实在shell的运行过程中,如果他使用了方法,他是会去开启一个子shell来进行运算的,案例如下
```bash
main(){
    echo $1
}
main
```
当前内容无法获取到`$1`

```bash
main(){
    echo $1
}
main $@
```
如上就可以,因为将所有的参数都传入到了数组当中去

```bash
export nums=1 # 环境变量

_stats() {
	local nums=1 # 局部变量
	echo ${nums}
}

nums=1 # 全局变量
```

## 按行遍历读取内容
`while read line;do;done < xx.txt`  其中每一行的数据就是`$line`

两种写法
```bash
while read line
do
       …
done < file


command | while read line
do
    …
done
```
read通过输入重定向，把file的第一行所有的内容赋值给变量line，循环体内的命令一般包含对变量line的处理；然后循环处理file的第二行、第三行。。。一直到file的最后一行。还记得while根据其后的命令退出状态来判断是否执行循环体吗？是的，read命令也有退出状态，当它从文件file中读到内容时，退出状态为0，循环继续惊醒；当read从文件中读完最后一行后，下次便没有内容可读了，此时read的退出状态为非0，所以循环才会退出


```bash
#!/bin/bash
sum=0
while read line
do
line_n=`echo ${line}|sed 's/[^0-9]//g'|wc -L`
echo ${line_n}
sum=$(($sum+$line_n))
done < $1
echo "sum:$sum"
```


## catEOF
```bash
# 覆盖写入
cat > xx.txt << EOF
> more  xx.txt
> helloworld
> helloworld
> helloworld
> xxxx
EOF

# 追加写入
cat >> xx.txt << EOF
> more  xx.txt
> helloworld
> helloworld
> helloworld
> xxxx
EOF
```


## 数值计算
数值就算有很多的工具,也有对应的不同方式,有些只支持整数运算,最全的就是bc这个第三方库
但是有些机器上面是没有bc的第三方库的,需要下载
```bash
yum install -y bc
```

bc支持任意精度的数值运算,使用情况如下
```bash
echo "2+2" | bc
echo "2.0+2.0" | bc
sum=$(echo "2+2" | bc)
sum=$(echo "2*2" | bc)
```

整数计算,有很多的工具只能支持简单的整数运算,且expr在格式上也有一定的要求,因为他还能处理简单的字符串处理

整数运算`$(())`,`$[]`,`expr 必须在运算符前后有空格,不然会被识别成为字符串`
举例,如果是写成`$(expr 1+1)` 结果是 `1+1`
举例,如果是写成`$(expr 1+ 1)` 结果是 `expr: syntax error` 报错,因为字符串和数值无法相加
```bash
sum=$((1+2))
sum=$[1+2]
sum=$(expr 1 + 1)
```

expr的字符串操作
```bash
# 提取指定字符串下标,只能提取一个字符的位置
ind=$(expr index "69lki" "9") # res 2
# 截取字符串子串
substring=$(expr substr "63lki","1",3) # res 63l
# 字符串长度
con=$(expr length "sdf") # res 3
```

## for循环的几种变式
```bash
# 语法
for   变量    in   {起始值..终止值}
do 
    循环主体
done
# 语法3
for   变量    in     `命令`
do
        循环主体
done
# 语法4
for   ((初始值;循环控制;变量变化))
do
        循环主体
done
```

## while循环
```bash
# 语法1
while  true;do
      循环主体
      break;
      continue;
done
```

## if判断
```bash
if [ -d xx.txt ];then echo "success";fi
if [ -d xx.txt ];then echo "success";else echo "fail";fi
[ -d xx.txt ] || action # 错则执行action
[ -d xx.txt ] && action # 对则执行action
```

## 调试
```bash
bash -vx ./xx.sh
```
在vim中使用shell

为了方便调试 可以在vim中使用shell 会自动保存，执行结束之后又可以按回车键回到vim当中去

`:!{cmd}` 是 Vim 命令，用来执行一条 Shell 命令，命令完成后按任意键回到 Vim
```bash
:!ifconfig en0
:!!
:!ansible 10.0.6.35 -m script  -a "xx.sh -z xx -qs
```



## shell引入python程序
```bash
#!/usr/bin/bash
/usr/bin/python << EOF
print("hello world")
EOF

# or
/usr/bin/python ./xx.py
```


## shell系统方法总结
shell脚本基本架构就是各类的方法,通过`usage`或者`tips`来做提示 通过`getops`做选项上的选择(只支持单个字符),通过`case`来做选项上的选择(可以做多个字符上的判断,使用$1 $2来判断,参数较少的时候适用)

### 读取版本
```bash
get_version(){
    if [[ -s /etc/redhat-release ]]; then
        grep -oE  "[0-9.]+" /etc/redhat-release
    else
        grep -oE  "[0-9.]+" /etc/issue
    fi
}


get_version
```

### 如果默认没有输入的参数,那么执行install
```bash
action=${1:-"install"}
相当于  [ -z $1 ] && $1="install"
```

### read

```bash

echo && read  -p "请输入数字 [1-3]：" menu_num
# 指定多个变量
read -p "Enter your name:" first last
# 当不存在指定变量名的时候,会放到REPLY这个环境变量当中去
read -p "Enter your name:" && echo $REPLY
# 对输入时间要求 ,单位秒
read -p  -t 1 "Enter your name:" 
# 对输入字符长度有要求
read -p  -n 1 "Enter your name:" 
# 对输入的字符隐藏
read -s -p "Enter your passwd: " pass
# 循环读取文本行
while read line;do echo ${line};done < xx.txt
```


### usage or tips
```bash
usage(){
cat << EOF
**************************************
*       thank you use                *
*       Author: hzjsea               *
*       Date: you guess              *
**************************************
EOF




cat << EOF
+-------------------------------------------------+
|  Thank you for usesing!                         |
+-------------------------------------------------+
EOF
}

```

### menus

```bash
index=-1

function show_menus(){
echo && cat << EOF
【System Environment】                            【Common Tools】
[1] Initialization System environment.            [4] Install Nginx1.6.
[2] Configure SSH_Auth.                           [5] Install Jdk1.7.
[3] Configure Network-bonding.                    [6] Install Maven3.3.9.

[100] Show Menus.
[0] Exit.
EOF
}

show_menus
while(($index!=0))
do
  echo 'Please input index:'
  read index
  case  $index  in
    100 ) show_menus
        ;;
    1 ) $filepath/base/sys.sh
        ;;
    2 ) $filepath/base/ssh.sh
        ;;
    3 ) $filepath/network/network-bonding.sh
        ;;
    4 ) $filepath/mongodb/install_2.6.sh
        ;;
    5 ) $filepath/mongodb/install_3.0.sh
        ;;
    6 ) $filepath/mysql/install.sh
        ;;
    * ) show_menus
        ;;
  esac
done
```

### 用拼接参数的形式来执行字符串形式的方法
```bash
action=${1:-"install"}

case ${action} in
    install|uninstall|update|start|stop|restart)
        do_${action}
        ;;
    status)
        do_${action}
        ;;
    script)
        check_script_update
        ;;
    uid)
        do_${action} 
        ;;
    *)
        usage
        ;;
esac

do_install(){}
do_unstall(){}
do_status(){}
do_uid(){}
```


### getopts

这是shell的一个内联函数，用来获取指定的参数
```bash
#!/bin/bash
# 模板
tips() {
    echo "
Use : xxxx
shortDesc : xxx
LongDesc : xxxx

Command: xxxx
    -n   add agrs
    -d   add args
    -z   add args
    -s   true or false
    -q   true or flase

Global Command : xxx
    -v
"
}

while getopts 'n:d:z:c:sqvm' options; do
    case ${options} in
    z) echo "option 当前参数内容 ${name}"+${OPTARG} ;;
    s) echo "is true" ;;
    q) echo "is q true" ;;
    n) echo $OPTIND ;;
    m) echo "$(($OPTIND - 1 ))";;
    v) echo "version is 0.1" ;;
    c) echo "$(($OPTIND - 1))" ;;
    *) tips ;;
    esac
done

```

getopts可以获取shell脚本后面加入的参数，并根据不同指令做出模式匹配
匹配的规则如下
- `n:` 表示 匹配 `-n xx`
- `s` 表示匹配 `-s` 我们把它叫做开关参数，就是当他存在的时候就为true

一般建议有参数的放在前面，没有参数的放在后面 
另外getopts不支持长参数，getopt支持，但是是第三方软件

为了方便定位参数，getopts 还提供两个内容
- `OPTARG` 表示指令后面的跟的参数  比如`-z xx` ,那么`echo ${OPTARG}` 的结果就是 xx
- `OPTIND` 表示参数个数+1 比如`-z xx` 那么`echo ${OPTIND}` 的结果就是 3 
所以匹配最后一个参数就是`$(($OPTIND-1))`


https://www.cnblogs.com/kevingrace/p/11753294.html


## shell模板
1. 参数比较少，脚本化，使用`$1 ~ $n` 来获取参数
 
```bash
#!/bin/bash

echo "helloworld"


if [ ! -z $1 ];then
    echo $1
else
    echo "no args"
fi
```

2. 参数比较多，需要制定多种餐宿，使用getops来完成，写一个简单地echo 来完成提示  项目化

```bash
frontcolor="\033[34m"
backcolor="\033[33m"

tips(){
echo -e"
${frontcolor}
Use: xxxx
shortDesc: xxx
LongDesc: xxxx

Command
    -n   add agrs
    -d   add args
    -z   add args
    -s   true or false
    -q   true or flase

Global Command
    -v
Example:
    ./xx.sh -n helloworld
${backcolor}
"
}

func(){
while getopts 'n:d:z:sqv' options;
do
    case ${options} in
        z)  echo "option 当前参数内容 ${name}"+${OPTARG};;
        s)  echo "is true";;
        q)  echo "is q true";;
        n)  echo $@ + $OPTIND;;
        v)  echo "version is 0.1";;
        *)  tips
    esac
done  
}


main(){
    if [ ! -z $1 ];then
        func $1
    else
        tips
    fi 
}

main $@
```

3. 工程化  有二级命令的时候，可以使用gocobra来使用



## shell_plugin
提供shell的一些代码块，可以实现某一块功能

### 转转转
循环提示
```bash
#!/usr/bin/env sh

nums=('-' '|' '\' '/')
i=0
for (( i; i < 4; i++ )); do
  printf "\r${nu
58.216.101.204
ms[i]}"
  if [ $i -eq 3 ]; then
    i=$((i-3))
  fi
done
```

### awk提取变量并输出
```bash
while read line;do
    echo $line | awk '{print $1,$2}'   | while read vlan vlanid;do
                                                echo $vlan $vlanid
                                         done
done < xx
```

### 子shell与父shell的变量传递问题

```bash

declare -A chia_farm_summary

chia_farm_summary=(
    [Farming_Status]=""
    [TXCH_Total_Chia_Farmed]=""
    [TXCH_Block_Rewards]=""
    [TXCH_User_Transaction_Fees]=""
    [Last_Height_Farmed]=""
    [Plot_Count]=""
    [Total_Size_of_Plots]=""
    [Total_Network_Space]=""
    [Expected_Time_to_Win]=""
)

# 管道后面是子shell 里面的赋值不会传递到父shell当中，所以父shell出来以后 字典依旧是默认值
# 这里管道后面一层就是一个子shell  前几层子shell都没关系，但是到了while那一层要赋值给父shell的变量了，子shell是无法改变父shell中的内容，所以跳出来之后又是默认值 ""
_chia_farm_summary_action(){
    /home/jh/chia-blockchain/venv/bin/chia farm summary | 
        sed 's/[[:space:]]//g' |
        awk -F ":" '$1 !~ /Note/{print $2}' |
        xargs |
            while read a b c d e f g h i; do
                chia_farm_summary[Farming_Status]=$a
                chia_farm_summary[TXCH_Total_Chia_Farmed]=$b
                chia_farm_summary[TXCH_Block_Rewards]=$c
                chia_farm_summary[TXCH_User_Transaction_Fees]=$d
                chia_farm_summary[Last_Height_Farmed]=$e
                chia_farm_summary[Plot_Count]=$f
                chia_farm_summary[Total_Size_of_Plots]=$g
                chia_farm_summary[Total_Network_Space]=$h
                chia_farm_summary[Expected_Time_to_Win]=$i
            done
}

## 解决办法就是使用一个临时文件过渡，让赋值的过程执行在父shell当中


_chia_farm_summary_action(){
    /home/jh/chia-blockchain/venv/bin/chia farm summary | 
        sed 's/[[:space:]]//g' |
        awk -F ":" '$1 !~ /Note/{print $2}' |
        xargs > ./temp.txt
        
    while read a b c d e f g h i; do
        chia_farm_summary[Farming_Status]=$a
        chia_farm_summary[TXCH_Total_Chia_Farmed]=$b
        chia_farm_summary[TXCH_Block_Rewards]=$c
        chia_farm_summary[TXCH_User_Transaction_Fees]=$d
        chia_farm_summary[Last_Height_Farmed]=$e
        chia_farm_summary[Plot_Count]=$f
        chia_farm_summary[Total_Size_of_Plots]=$g
        chia_farm_summary[Total_Network_Space]=$h
        chia_farm_summary[Expected_Time_to_Win]=$i
    done < ./temp.txt
}



```