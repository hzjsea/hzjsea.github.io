---
title: 手动编写bash-completion
categories:
  - shell
tags:
  - shell
abbrlink: 6a6a56fc
date: 2020-03-25 11:37:02
---
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [手动编写bash-completion](#手动编写bash-completion)
  - [准备阶段](#准备阶段)
    - [complete命令](#complete命令)
    - [compgen 筛选命令](#compgen-筛选命令)
    - [compopt 修改补全命令设置](#compopt-修改补全命令设置)
    - [内置补全命令](#内置补全命令)
  - [构建一个自动完成的替补工具](#构建一个自动完成的替补工具)
    - [自动补全模板](#自动补全模板)
  - [构建一个文本筛选系统](#构建一个文本筛选系统)
  - [更高级的写法](#更高级的写法)
  - [自动完成脚本实例](#自动完成脚本实例)
  - [sws实例](#sws实例)

<!-- /code_chunk_output -->
<!-- more -->
# 手动编写bash-completion
bash提供了一个complete内建命令，它的用途是规定参数怎么自动补全。 有了它，第三方开发的命令就可以根据自己的实际情况指定自动提示功能了！
具体的，我们可以用man来查看这个内建命令
```bash
man complete
```
同样的，我们也可以编写自己的补全命令脚本，通过bash_completion这个包来完成，
首先安装这个包
```bash
yum install bash-completion -y
```
安装完成之后，我们来看一下他安装的目录环境
```bash
[root@node bash_completion.d]# rpm -ql bash-completion
/etc/bash_completion.d
/etc/bash_completion.d/redefine_filedir
/etc/profile.d/bash_completion.sh
/usr/share/bash-completion
```
其中`/etc/bash_completion.d/`下面是用来放这些自动化补全脚本的， 然后通过`/etc/profile.d/bash_completion.sh`这个脚本来引入，其中/etc/profile.d下面的sh脚本是对系统中的所有用户都适用的
,并且在每次登录的时候都会加载

然后这个`redefine_filedir`貌似是一个读取文件的脚本
然后这个`bash_completion`会去读取/etc/bash_completion.d/下面的脚本文件
```bash
  # Check for interactive bash and that we haven't already been sourced.
  [ -z "$BASH_VERSION" -o -z "$PS1" -o -n "$BASH_COMPLETION_COMPAT_DIR" ] && return

  # Check for recent enough version of bash.
  bash=${BASH_VERSION%.*}; bmajor=${bash%.*}; bminor=${bash#*.}
  if [ $bmajor -gt 4 ] || [ $bmajor -eq 4 -a $bminor -ge 1 ]; then
      [ -r "${XDG_CONFIG_HOME:-$HOME/.config}/bash_completion" ] && \
          . "${XDG_CONFIG_HOME:-$HOME/.config}/bash_completion"
      if shopt -q progcomp && [ -r /usr/share/bash-completion/bash_completion ]; then
          # Source completion code.
          . /usr/share/bash-completion/bash_completion
      fi
  fi
  unset bash bmajor bminor
```


## 准备阶段
首先完成这个自动补全命令需要一些其他命令来做补充

### complete命令
这个命令是一个补全命令
```bash
[root@node ~]# help complete
complete: complete [-abcdefgjksuv] [-pr] [-DE] [-o option] [-A action] [-G globpat] [-W wordlist]  [-F function] [-C command] [-X filterpat] [-P prefix] [-S suffix] [name ...]
    Specify how arguments are to be completed by Readline.
    指明
    For each NAME, specify how arguments are to be completed.  If no options
    are supplied, existing completion specifications are printed in a way that
    allows them to be reused as input.

    Options:
      -p        print existing completion specifications in a reusable format
      -r        remove a completion specification for each NAME, or, if no
        NAMEs are supplied, all completion specifications
      -D        apply the completions and actions as the default for commands
        without any specific completion defined
      -E        apply the completions and actions to "empty" commands --
        completion attempted on a blank line

    When completion is attempted, the actions are applied in the order the
    uppercase-letter options are listed above.  The -D option takes
    precedence over -E.

    Exit Status:
    Returns success unless an invalid option is supplied or an error occurs.
```

### compgen 筛选命令
用来筛选生成匹配的候选补全结果
就是用来
```bash
[root@node ~]# help compgen
compgen: compgen [-abcdefgjksuv] [-o option]  [-A action] [-G globpat] [-W wordlist]  [-F function] [-C command] [-X filterpat] [-P prefix] [-S suffix] [word]
    Display possible completions depending on the options.

    Intended to be used from within a shell function generating possible
    completions.  If the optional WORD argument is supplied, matches against
    WORD are generated.

    Exit Status:
    Returns success unless an invalid option is supplied or an error occurs.
```
`- W wordlist` 分割wordlist中的单词，生成候选补全列表
`compgen -w "aa ab ac ad ae" -- "a"` 这个表示从"aa ab ac ad ae"中匹配出"a"开头的单词


### compopt 修改补全命令设置
用于修改补全命令设置，这个命令必须在补全函数中使用，否则会报错。
```bash
[root@node ~]# help compopt
compopt: compopt [-o|+o option] [-DE] [name ...]
    Modify or display completion options.

    Modify the completion options for each NAME, or, if no NAMEs are supplied,
    the completion currently being executed.  If no OPTIONs are given, print
    the completion options for each NAME or the current completion specification.

    Options:
        -o option       Set completion option OPTION for each NAME
        -D              Change options for the "default" command completion
        -E              Change options for the "empty" command completion
```

`+o option 启用option配置`
`-o option 弃用option配置`

### 内置补全命令
|variable   | description   |
|---|---|
| COMP_WORDS |  类型为数组，存放当前命令行中输入的所有单词  |
| COMP_CWORD |   类型为整数，当前输入的单词在COMP_WORDS中的索引 |
| COMPREPLY |  类型为数组，候选的补全结果  |
| COMP_WORDBREAKS |  类型为字符串，表示单词之间的分隔符  |
| COMP_LINE |  类型为字符串，表示当前的命令行输入字符  |
| COMP_POINT |  类型为整数，表示光标在当前命令行的哪个位置  |



## 构建一个自动完成的替补工具

自动补全脚本
```bash
# vi /etc/completion.d/ws
_start(){
    # 数组，又来收集提醒的内容
    COMPREPLY=()

    # 返回当前输入的参数
    local cur=${COMP_WORDS[COMP_CWORD]}
    # 返回当前输入的
    local cmd=${COMP_WORDS[COMP_CWORD-1]}
    case $cmd  in 
        'ws')
        COMPREPLY=($(compgen -W '--word1 --word2 --word3' -- $cur))
        ;;
        '--word1')
        COMPREPLY=($(compgen -W '--exec1 --exec2 --exec3' -- $cur))
        ;;
    esac
    return 0
    
}
complete -F _start ws
```
启动命令 ，或者重新登录一下，反正放在这个文件夹下面的脚本都会在用户登录的时候去加载
`source ws`

使用命令
```bash
ws [TAb][TAb] #就会有提示出来了
```


### 自动补全模板
```bash
# vi /etc/completion.d/{{name}}
_start(){
    # 数组，又来收集提醒的内容
    COMPREPLY=()
    # 返回当前输入的参数
    local cur=${COMP_WORDS[COMP_CWORD]}
    # 返回当前输入的
    local cmd=${COMP_WORDS[COMP_CWORD-1]}
    case $cmd  in 
        'arg1')
        COMPREPLY=($(compgen -W '{{argument}}' -- $cur))
        ;;
        'arg2')
        COMPREPLY=($(compgen -W '{{argument}}' -- $cur))
        ;;
    esac
    return 0
    
}
complete -F _start {{name}}
```


## 构建一个文本筛选系统
同Upai里面的一个sws 自动补全工具
```bash
# 创建过滤文本 类似如下
[EDGE-SWITCH]
ZEN-CN-HKG-S01 ansible_ssh_=ip_host_example T1/0/22
CMN-SC-CTU4-S01 ansible_ssh_=ip_host_example T1/0/48
CUN-FJ-XMN-S01 ansible_ssh_=ip_host_example uplink=T2/0/49
CUN-SD-DEZ-S01 ansible_ssh_=ip_host_example uplink=T1/0/49
CTN-SN-XIY-S01 ansible_ssh_=ip_host_example uplink=B100
CTN-JS-TAZ-S01 ansible_ssh_=ip_host_example uplink=B100
GWN-BJ-PEK-S01 ansible_ssh_=ip_host_example uplink=T1/0/24
CTN-FJ-FOC-S01 ansible_ssh_=ip_host_example uplink=B110
CMN-JS-CZX-S01 ansible_ssh_=ip_host_example uplink=B100
CMN-JS-CZX-S02 ansible_ssh_=ip_host_example uplink=B100
CTN-ZJ-JGH-S01 ansible_ssh_=ip_host_example uplink=T1/0/49
CMN-HA-CGO1-S01 ansible_ssh_=ip_host_example uplink=T1/0/46,T1/0/47,T1/0/48
CTN-GD-JIM2-S01 ansible_ssh_=ip_host_example uplink=B100
CTN-LN-ENX-S01 ansible_ssh_=ip_host_example uplink=T1/0/49
CUN-ZJ-HUZ-S01 ansible_ssh_=ip_host_example uplink=T1/0/52
GWN-SH-SHA-S01 ansible_ssh_=ip_host_example uplink=T1/0/48
#CTN-SN-TOC-S01 ansible_ssh_=ip_host_example uplink=B110
NTT-US-LAX-S01 ansible_ssh_=ip_host_example uplink=B100
CMN-SD-TNA-S01 ansible_ssh_=ip_host_example uplink=B130
CMN-GD-CAN-S01 ansible_ssh_=ip_host_example uplink=B1
GTT-DE-FRA-S01 ansible_ssh_=ip_host_example uplink=T1/0/49
SGT-SG-SIN-S01 ansible_ssh_=ip_host_example uplink=T1/0/49
CUN-SD-LYI-S02 ansible_ssh_=ip_host_example uplink=B200
PCW-CN-HKG-S01 ansible_ssh_=ip_host_example uplink=T1/0/48
NTT-CN-HKG-S01 ansible_ssh_=ip_host_example uplink=T1/0/48
NTT-CN-HKG-S02 ansible_ssh_=ip_host_example uplink=T1/0/52
```
创建自动补全脚本
```bash
  # vi /etc/completion.d/sws
  target_file="/root/hzj/pro_shell/sws/inventory"
  _start(){
      # 数组，用来收集提醒的内容
      COMPREPLY=()
      # 返回当前输入的参数
      local cur=${COMP_WORDS[COMP_CWORD]}
      # 返回当前输入的
      local cmd=${COMP_WORDS[COMP_CWORD-1]}
  ▏       ▏       # 做参数整理
  ▏         cur=$(echo $cur |tr '[:lower:]' '[:upper:]'| tr '.' '-')
  ▏       ▏       #输出参数
  ▏       ▏       if [ ${cur}  ]
  ▏       ▏       then
  ▏       ▏       ▏       machines=$(grep -e "^${cur}" $target_file )
  ▏       ▏       ▏       COMPREPLY=($(compgen -W '$machines' -- $cur))
  ▏       ▏       ▏       return 0
  ▏       ▏       fi

  }
  complete -F _start sws
```

创建脚本
```bash
echo 'echo echo "hello world"' > /hzj/pro_shell/sws
```


## 更高级的写法

如果你用的是zsh，那么补全命令应该在/root/.zsh/completion/下面
上面只是一些小小的个例写法，另外高级的可以抛弃他给的那些内建函数，自己写,比如upssh




## 自动完成脚本实例
```bash
#!/bin/bash

_start(){
    # 数组，又来收集提醒的内容
    COMPREPLY=()

    local cur=${COMP_WORDS[COMP_CWORD]}
    echo $cur
    local cmd=${COMP_WORDS[COMP_CWORD-1]}
    echo $cmd
    case $cmd  in 
        './06_sws.sh')
        COMPREPLY=($(compgen -W '--word1 --word2 --word3' -- $cur))
        ;;
        '--word1')
        COMPREPLY=($(compgen -W '--exec1 --exec2 --exec3' -- $cur))
        ;;
    esac
    return 0
    
}
complete -F _start ./06_sws.sh
    
```

## sws实例
```bash
#!/bin/bash
usage(){
echo  "Usage:  -p port [ default: 22 ]
        -u user [ default: admin ]
        <-i> [ default: /root/.ssh/swmgmt ]"
exit 1
}

while getopts "p:u:i:" OPT > /dev/null 2>&1
    do
        case $OPT in
            p)
             PORT=$OPTARG
              ;;
            u)
              USER=$OPTARG
              ;;
            i)
              IDY_FILE=$OPTARG
              ;;
            *)
              usage
              ;;
         esac
done


shift $(($OPTIND - 1))


# 匹配IP
if [ $@ ]
   then
      IP=${@#*/}
else
    usage
fi

# 匹配端口
PORT=${PORT:=22}

# 匹配用户
USER=admin

# 匹配秘钥
IDY_FILE=${IDY_FILE:=/root/.ssh/swmgmt}



OPTIONS="-p ${PORT}"

[ -e ${IDY_FILE} ] && OPTIONS="${OPTIONS} -i ${IDY_FILE}"

ssh ${OPTIONS} ${USER}@${IP}
```