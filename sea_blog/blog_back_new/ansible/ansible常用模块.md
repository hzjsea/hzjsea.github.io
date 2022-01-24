---
title: ansible常用模块
date: 2020-11-12 15:25:01
categories: [ansible,]  
tags: [ansible,]
---


<!-- more -->

# ansible常用模块
因为ansible模块有很多，但是常用的就那么几个，像yum这种完全可以写在脚本中然后用script模块来完成



## script模块
这个模块会把本地的脚本文件传到远端的/tmp/目录下面然后运行，完成之后会删除脚本
```bash
ansible 192.168.1.3 -m script -a '/root/ansible_test.sh'
```


## shell模块与command模块
shell模块和command模块都可以用来执行shell命令在远端主机上，但是存在一定的区别
- ommand 模块命令将不会使用 shell 执行. 因此, 像 $HOME 这样的变量是不可用的。还有像`<, >, |, ;, &`都将不可用。
- shell 模块通过shell程序执行， 默认是`/bin/sh`, `<, >, |, ;, &` 可用。但这样有潜在的 shell 注入风险， 后面会详谈.
- command 模块更安全，因为他不受用户环境的影响。 也很大的避免了潜在的 shell 注入风险.

当command使用管道符或逻辑符的时候，会出现如下的内容
```bash
10.0.6.35 | FAILED | rc=1 >>
error: garbage option

Usage:
 ps [options]

 Try 'ps --help <simple|list|output|threads|misc|all>'
  or 'ps --help <s|l|o|t|m|a>'
 for additional help text.

For more details see ps(1).non-zero return code
```
但是shell就可以成功执行，所以在后续的使用中要考虑到这个