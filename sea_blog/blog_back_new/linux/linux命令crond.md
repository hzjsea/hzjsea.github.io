---
title: linux命令crond
date: 2020-12-09 14:19:42
categories: [linux,]  
tags: [linuxcc,linux]
---


<!-- more -->
# linux命令crond

如果没有crond 或者 crontab命令,则说明可能使用的是精简过得系统,可以使用下面的命令安装
```bash
yum install crontabs -y
```

- crontab 是针对当前用户的
- crond 是针对全局的

> 注意

所有文件都需要指定绝对路径,不能使用相对路径
`*/1 * * * * echo 'xxx' >>  ./xxx.log` 这个task只会更新到/root/下面


> 注意2

所有crond控制的脚本都必须是有绝对路径的 不然还是会跑到root下面 包括所有的命令路径
比如我要备份数据库,我会使用到mysqldump 因此我要使用绝对路径来使用mysqldump `/usr/local/sbin/mysqldump`

```bash
case $i in
        init)
                echo "1" >> ./xx.txt
                ;;
        *)
                echo "default" >> ./xx.txt  # 这里没有写绝对路径,就会跑到/root/下面去了
                ;;
esac

time_now=`date`
echo ${time_now} >> ./xx.txt

crond如下
*/1 * * * *  root sh /etc/cron.d/xx.sh init >> /etc/cron.d/xxx.txt
```

> 使用crontab

```bash
# 在crontab -e 创建的文件下面
>> crontab -e
*/1 * * * * echo 'xxx' >>  ./xxx.log
```

> 使用crond

```bash
#在/etc/crond 文件夹下面
*/1 * * * *  root echo 'xxx' >>  ./xxx.log
```

## 时间分布图
![2020-12-09-14-26-13](http://noback.upyun.com/2020-12-09-14-26-13.png)

举例
```bash
# 每分钟写内容到log当中
*/1 * * * *  root echo 'xxx' >>  ./xxx.log
# 每60分钟写内容到log当中
*/60 * * * *  root echo 'xxx' >>  ./xxx.log
```