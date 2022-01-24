---
title: mysql主从备份
categories:
  - mysql
tags:
  - mysql
abbrlink: 23676c42
date: 2020-06-05 15:10:44
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [mysql主从复制](#mysql主从复制)
  - [基于二进制日志点的异步复制解决方案](#基于二进制日志点的异步复制解决方案)
  - [主从同步进阶版--读写分离](#主从同步进阶版-读写分离)
  - [master和salve数据库的数据不一致解决方案](#master和salve数据库的数据不一致解决方案)
    - [mysqldump 冷备份](#mysqldump-冷备份)
    - [Xtrabackup 热备份](#xtrabackup-热备份)
      - [备份流程](#备份流程)
      - [安装xtrabackup](#安装xtrabackup)
      - [使用xtrabackup全量备份](#使用xtrabackup全量备份)
      - [使用xtrabackup增量备份](#使用xtrabackup增量备份)
      - [xtrabackup 流式备份](#xtrabackup-流式备份)
      - [备份恢复](#备份恢复)
      - [区别](#区别)
  - [xtrabackup详解](#xtrabackup详解)
  - [错误](#错误)
    - [docker mysql](#docker-mysql)
    - [innobackupex 命令出错](#innobackupex-命令出错)
    - [docker-mysql无法被innobackupex备份](#docker-mysql无法被innobackupex备份)
  - [docker-mysql重启数据库](#docker-mysql重启数据库)
  - [docker-mysql运行](#docker-mysql运行)
  - [mysql 一键脚本](#mysql-一键脚本)
  - [extra](#extra)

<!-- /code_chunk_output -->
<!-- more -->


# mysql主从复制
mysql主从复制的两种解决方案
1. 传统方式：
mysql支持单向、异步复制，复制过程中一个服务器充当主服务器，而一个或多个其它服务器充当从服务器。mysql复制基于主服务器在二进制日志中跟踪所有对数据库的更改(更新、删除等等)。因此，要进行复制，必须在主服务器上启用二进制日志。每个从服务器从主服务器接收主服务器已经记录到其二进制日志的保存的更新。当一个从服务器连接主服务器时，它通知主服务器从服务器在日志中读取的最后一次成功更新的位置。从服务器接收从那时起发生的任何更新，并在本机上执行相同的更新。然后封锁并等待主服务器通知新的更新。从服务器执行备份不会干扰主服务器，在备份过程中主服务器可以继续处理更新。
基于主库的bin-log将日志事件和事件位置复制到从库，从库再加以应用来达到主从同步的目的。
2. Gtid方式（MySQL>=5.7推荐使用）：
基于GTID的复制中，从库会告知主库已经执行的事务的GTID的值，然后主库会将所有未执行的事务的GTID的列表返回给从库，并且可以保证同一个事务只在指定的从库执行一次。

mysql复制的几种方式
1. 异步复制
一个主库，一个或多个从库，数据异步同步到从库。
2. 同步复制
在MySQL cluster中特有的复制方式。
3. 半同步复制
在异步复制的基础上，确保任何一个主库上的事物在提交之前至少有一个从库已经收到该事物并日志记录下来。
4. 延迟复制
在异步复制的基础上，人为设定主库和从库的数据同步延迟时间，即保证数据延迟至少是这个参数。

mysql复制的几种形式
1. **基于语句的复制**：在主服务器上执行的SQL语句，在从服务器上执行同样的语句。MySQL默认采用基于语句的复制，效率比较高。 一旦发现没法精确复制时，会自动选着基于行的复制
```bash
binlog_format=Statement
```
2. **基于行的复制**：把改变的内容复制过去，而不是把命令在从服务器上执行一遍. 从mysql5.0开始支持
```bash
binlog_format=Row
```
3. **混合类型的复制**：默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，就会采用基于行的复制。
```bash
binlog_format=Mixed
```

## 基于二进制日志点的异步复制解决方案

我们在MySQL中配置了主从之后，
- 只要我们对Master节点进行了写操作，这个操作将会被保存到MySQL的binary-log（bin-log）日志当中， 这些记录叫做二进制日志事件，binary log events）
- slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)
- slave 重做中继日志中的事件，将改变反映它自己的数据。

![2020-06-08-08-51-22](http://noback.upyun.com/2020-06-08-08-51-22.png)

两个线程的主要作用
- I/O线程：该线程链接到master机器，master机器的binlog发送到slave的时候，IO线程会将该日志内容写在本地的中继日志（Relay log）中。
- SQL线程：该线程读取中继日志中的内容，并且根据中继日志中的内容对Slave数据库做相应的操作。
可能造成的问题：在写请求相当多的情况下，可能会造成Slave数据和Master数据不一致的情况，这是因为日志传输过程中的短暂延迟、或者写命令较多，系统速度不匹配造成的。
这大致就是MySQL主从同步的原理，真正在其中起到作用的实际上就是这两个日志文件，binlog和中继日志（Relay log）。

创建两个mysql容器 因为配置文件我们后面要做修改，所以需要外面挂载进去
设置配置文件
```bash
mkdir /hzj/mysql
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# ====== 主从复制配置文件

server-id = 1            # 服务器 id 号，不要和其他服务器重复
log-bin=mysql-bin     # 开启二进制日志
log_bin_index = mysql-bin.index    # 索引二进制日志的文件名
sync_binlog = 1      # 设为1就是把MySql每次发生的修改和事件的日志即时同步到硬盘上
binlog_format = Row     # 复制模式 Statement, Row, mixed 这里是基于行的复制
skip_slave_start = 1     # 防止从服务器在崩溃后自动开启，以给你足够的时间修复。
max_binlog_size = 200M     # 指定二进制日志的大小

binlog-do-db = datafordata
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup
sql-mode=NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid


# slave.cnf
[mysqld]
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# ======= 主从赋值配置文件
server-id = 2  # 服务器 id 号，不要和其他服务器重复
read_only = 1  # 让从服务器只读，可以防止有人误从服务器插入数据，导致主从数据不一致。·
log-bin=mysql-bin # 开启二进制日志
log_bin_index = mysql-bin.index# 索引二进制日志的文件名
log_slave_updates = 1
relay_log = mysql-relay-bin  # 中继日志·
relay_log_index = mysql-relay-bin.index
skip_slave_start = 1  # 防止从服务器在崩溃后自动开启，以给你足够的时间修复。·
max_binlog_size = 200M   # 指定二进制日志的大小

binlog-do-db = datafordata
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup
sql-mode=NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# 1. 主从备份相关配置 - 从服务器
# 不在这里配置主服务器信息，而是通过命令来配置
server-id = 2 # 服务器 id 号，不要和其他服务器重复
read_only = 1 # 让从服务器只读，可以防止有人误从服务器插入数据，导致主从数据不一致。·
log-bin=mysql-bin # 开启二进制日志
log_bin_index = mysql-bin.index# 索引二进制日志的文件名
log_slave_updates = 1
relay_log = mysql-relay-bin # 中继日志·
relay_log_index = mysql-relay-bin.index
skip_slave_start = 1 # 防止从服务器在崩溃后自动开启，以给你足够的时间修复。·
max_binlog_size = 200M  # 指定二进制日志的大小

# 以下配置是为了方便以后，从库切换为主库
# 1.1 需要同步的二进制数据库名
binlog-do-db = datafordata
# 1.2 不同步的二进制数据库名,如果不设置可以将其注释掉
# binlog-ignore-db = information_schema
# binlog-ignore-db = mysql

```
```bash
docker pull mysql:5.7
docker run -p 3305:3306 --name mysql-slave -v /root/hzj/hzj/mysql/slave.cnf:/etc/mysql/conf.d/slave.cnf -e MYSQL_ROOT_PASSWORD=upyun123 -itd mysql:5.7
docker run -p 3306:3306 --name mysql-master -v /root/hzj/hzj/mysql/master.cnf:/etc/mysql/conf.d/master.cnf -e MYSQL_ROOT_PASSWORD=upyun123 -itd mysql:5.7
```

进入容器配置远程用户
```bash
# 主机
docker exec -it mysql-master /bin/bash

> mysql -u root -p
# 修改本地root密码
> ALTER USER 'root'@'localhost' IDENTIFIED BY 'upyun123';
# 创建远程登录用户
> CREATE USER 'hzj'@'%' IDENTIFIED WITH mysql_native_password BY 'upyun123';
> GRANT ALL PRIVILEGES ON *.* TO 'hzj'@'%';

# 备机
docker exec -it mysql-slave /bin/bash

> mysql -u root -p
# 修改本地root密码
> ALTER USER 'root'@'localhost' IDENTIFIED BY 'upyun123';
# 创建远程登录用户
> CREATE USER 'hzj'@'%' IDENTIFIED WITH mysql_native_password BY 'upyun123';
> GRANT ALL PRIVILEGES ON *.* TO 'hzj'@'%';
```
![2020-06-05-15-30-18](http://noback.upyun.com/2020-06-05-15-30-18.png)

初始化主机上的内容，并插入内容，模拟需要备份的内容
```sql

create database datafordata  character set utf8mb4;

use datafordata;

CREATE TABLE `t_temp` (
	`id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
	`comment` varchar(10) NOT NULL DEFAULT '' COMMENT '备注',
	`create_time` bigint(20) NOT NULL ,
	`update_time` bigint(20) NOT NULL ,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;

# 插入数据
INSERT INTO `datafordata`.`t_temp` (`id`, `comment`, `create_time`, `update_time`) VALUES ('1', 'helloworld', '1', '1');
```
![2020-06-05-15-39-52](http://noback.upyun.com/2020-06-05-15-39-52.png)

更新完毕以后，查看主从机器中主机的状态
![2020-06-05-17-02-43](http://noback.upyun.com/2020-06-05-17-02-43.png)
![2020-06-05-18-21-19](http://noback.upyun.com/2020-06-05-18-21-19.png)

参数介绍
- File: mysql-bin.000033 #当前记录的日志
- Position: 337523 #日志中记录的位置  
- Binlog_Do_DB: 
- Binlog_Ignore_DB: 

在从机上设置同步目标
```bash
change  master to master_host='10.0.6.55',MASTER_PORT=3306,master_user='root',master_password='upyun123',master_log_file='mysql-bin.000003',master_log_pos=4210;
```
其中master_log_pos和master_log_file是在主机上面使用`show master status ` 查看对应的log名字和log大小
开启同步
```bash
start slave;
# 查看同步状态
show slave status\G;
#
```
![2020-06-05-18-28-14](http://noback.upyun.com/2020-06-05-18-28-14.png)
这里面如果是yes表示正在同步了，如果是no则表示没有同步

<font color='red'>主从备份的时候要保证两个数据库的数据结构是一一致的</font>


## 主从同步进阶版--读写分离
![2020-06-08-08-53-53](http://noback.upyun.com/2020-06-08-08-53-53.png)


## master和salve数据库的数据不一致解决方案
主从数据库的数据要保持一致，不然主从同步会出现bug..
如果主数据库中已经存在数据，有两种解决方案.
- 第一种方案是选择忽略主库之前的数据，不做处理。这种方案只适用于不重要的可有可无的数据，并且业务上能够容忍主从库数据不一致的场景。
- 第二种方案是对主库的数据进行备份，然后将主数据库中导出的数据导入到从数据库，然后再开启主从复制，以此来保证主从数据库数据一致

###  mysqldump 冷备份
在之前就备份好数据库，使用mysqldump备份数据库
mysqldump是mysql原生自带很好用的逻辑备份工具
备份的基本流程如下
```bash
1.调用FTWRL(flush tables with read lock)，全局禁止读写
2.开启快照读，获取此时的快照(仅对innodb表起作用)
3.备份非innodb表数据(*.frm,*.myi,*.myd等)
4.非innodb表备份完毕后，释放FTWRL锁
5.逐一备份innodb表数据
6.备份完成
```


常用备份参数
```bash
-A #备份全库
-B #备某一个数据库下的所有表
-R #备份存储过程和函数数据
--triggers # 备份触发器数据
--master-data = {1|2} # 告诉你备份后时刻的binlog位置 ,如果等于1，则将其打印为CHANGE MASTER命令; 如果等于2，那么该命令将以注释符号为前缀。
--single-transaction # 对innodb引擎进行热备
-F, --flush-logs # 刷新binlog日志
-x, --lock-all-tables # 锁定所有数据库的所有表。这是通过在整个转储期间采用全局读锁来实现的
-l, --lock-tables # 锁定所有表以供读取
-d,  # 仅表结构
-t, # 仅数据
--compact #减少无用数据输出(调试)

mysqldump -A -R --triggers --master-data=2 --single-transaction |gzip >/opt/all_$(date +%F).sql.gz 
```

```bash
# 1.锁定数据库，保证在同步的过程中不会有新的数据插入
flush tables with read lock;
# 2. 使用mysqldump备份数据库

[root@localhost data]# cd /data/db_backup/
[root@localhost db_backup]#  mysqldump -uroot -p --master-data=1 --single-transaction --routines --triggers --events  --all-databases > all.sql
Enter password

# 3.解除主数据库的锁定
unlock tables;

# 4. 把数据全量导入slave数据库，保证主从数据一致
```


###  Xtrabackup 热备份
Xtrabackup是一个开源的免费的热备工具，在Xtrabackup包中主要有Xtrabackup和innobackupex两个工具。xtrabackup 是用来备份 InnoDB 表的，不能备份非 InnoDB 表，和 mysqld server 没有交互；innobackupex 脚本用来备份非 InnoDB 表，同时会调用 xtrabackup 命令来备份 InnoDB 表，还会和 mysqld server 发送命令进行交互，如加读锁（FTWRL）、获取位点（SHOW SLAVE STATUS）等。简单来说，innobackupex 在 xtrabackup 之上做了一层封装。

安装完成xtarbackup之后，会有下面4个工具
```bash
usr
├── bin
│   ├── innobackupex  # 备份innodb表
│   ├── xbcrypt  # 备份加密
│   ├── xbstream  # 流式备份
│   └── xtrabackup  # 备份非innodb表
``` 

#### 备份流程
![2020-06-08-10-54-05](http://noback.upyun.com/2020-06-08-10-54-05.png)
备份过程
![2020-06-08-10-55-13](http://noback.upyun.com/2020-06-08-10-55-13.png)
恢复过程
![2020-06-08-10-44-07](http://noback.upyun.com/2020-06-08-10-44-07.png)


1. innobackupex 在启动后，会先 fork 一个进程，启动 xtrabackup进程，然后就等待 xtrabackup 备份完 ibd 数据文件；
2. xtrabackup 在备份 InnoDB 相关数据时，是有2种线程的，1种是 redo 拷贝线程，负责拷贝 redo 文件，1种是 ibd 拷贝线程，负责拷贝 ibd 文件；redo 拷贝线程只有一个，在 ibd 拷贝线程之前启动，在 ibd 线程结束后结束。xtrabackup 进程开始执行后，先启动 redo 拷贝线程，从最新的 checkpoint 点开始顺序拷贝 redo 日志；然后再启动 ibd 数据拷贝线程，在 xtrabackup 拷贝 ibd 过程中，innobackupex 进程一直处于等待状态（等待文件被创建）
3. xtrabackup 拷贝完成idb后，通知 innobackupex（通过创建文件），同时自己进入等待（redo 线程仍然继续拷贝）;
4. innobackupex 收到 xtrabackup 通知后，执行FLUSH TABLES WITH READ LOCK (FTWRL)，取得一致性位点，然后开始备份非 InnoDB 文件（包括 frm、MYD、MYI、CSV、opt、par等）。拷贝非 InnoDB 文件过程中，因为数据库处于全局只读状态，如果在业务的主库备份的话，要特别小心，非 InnoDB 表（主要是MyISAM）比较多的话整库只读时间就会比较长，这个影响一定要评估到。
5. 当 innobackupex 拷贝完所有非 InnoDB 表文件后，通知 xtrabackup（通过删文件） ，同时自己进入等待（等待另一个文件被创建）；
6. xtrabackup 收到 innobackupex 备份完非 InnoDB 通知后，就停止 redo 拷贝线程，然后通知 innobackupex redo log 拷贝完成（通过创建文件）；
7. innobackupex 收到 redo 备份完成通知后，就开始解锁，执行 UNLOCK TABLES；
8. 最后 innobackupex 和 xtrabackup 进程各自完成收尾工作，如资源的释放、写备份元数据信息等，innobackupex 等待 xtrabackup 子进程结束后退出。

在上面描述的文件拷贝，都是备份进程直接通过操作系统读取数据文件的，只在执行 SQL 命令时和数据库有交互，基本不影响数据库的运行，在备份非 InnoDB 时会有一段时间只读（如果没有MyISAM表的话，只读时间在几秒左右），在备份 InnoDB 数据文件时，对数据库完全没有影响，是真正的热备。

InnoDB 和非 InnoDB 文件的备份都是通过拷贝文件来做的，但是实现的方式不同，前者是以page为粒度做的(xtrabackup)，后者是 cp 或者 tar 命令(innobackupex)，xtrabackup 在读取每个page时会校验 checksum 值，保证数据块是一致的，而 innobackupex 在 cp MyISAM 文件时已经做了flush（FTWRL），磁盘上的文件也是完整的，所以最终备份集里的数据文件都是写入完整的。

 --- http://mysql.taobao.org/monthly/2016/03/07/


#### 安装xtrabackup
- 下载安装包
网址是https://www.percona.com/downloads/ 我这里使用的是2.4.14版本。
- 下载依赖包
然后下载依赖包，网址是http://rpmfind.net/linux/atrpms/el6-x86_64/atrpms/stable/libev-4.04-2.el6.x86_64.rpm。

```bash
mdkir /root/package && cd /root/package
wget Percona-XtraBackup-2.4.14-ref675d4-el7-x86_64-bundle.tar
wget http://rpmfind.net/linux/atrpms/el6-x86_64/atrpms/stable/libev-4.04-2.el6.x86_64.rpm

rpm -ivh libev-4.04-2.el6.x86_64.rpm
tar -xf Percona-XtraBackup-2.4.14-ref675d4-el7-x86_64-bundle.tar
yum -y install percona-xtrabackup-24-2.4.14-1.el7.x86_64.rpm
```

<font color='red'>xtrabackup是根据数据库的配置文件来查找数据库的，默认情况下指向/etc/my.cnf</font>

#### 使用xtrabackup全量备份
在备份数据库时会涉及到两个用户：系统用户与数据库内部的用户。
　　系统用户需要在datadir（配置文件内设置的目录）上具有读写执行权限（rwx）。而数据库内部用户需要：RELOAD和LOCK TABLES权限，为了执行FLUSH TABLES WITH READ LOCK；REPLICATION CLIENT权限，为了获取binary log（二进制日志文件）位置；CREATE TABLESPACE权限，为了导入表，用户表级别的恢复；SUPER权限，为了在slave环境下备份用来启动和关闭slave线程。

常用参数
```bash
--user=[数据库用户]    以什么用户身份进行操作
--password=[密码]    数据库用户的密码
--port=[端口号]    数据库的端口号，默认3306
--stream=[tar|xbstream]　　打包（数据流）
--defaults-file=[配置文件]　　指定默认配置文件，默认读取/etc/my.cnf
--no-timestamp　　不创建时间戳文件，而改用目的地址（可以自动创建）
--copy-back　　备份还原的主要选项
--incremental　　使用增量备份，默认使用的完整备份
--apply-log 应用数据
--incremental-basedir=[地址]　　与--incremental选项联合使用，该参数指定上一级备份的地址来做增量备份
```

使用
```bash
# 复制数据库中的内容
innobackupex --user=root --password=123456 ./db_backup/  # 会打印每一个备份的步骤
innobackupex --user=root --password=123456 ./db_backup/ 2>>./db_backup/backup.log　　# 将输出内容丢到黑洞如果不想要输出信息，可以将输出信息重定向到指定文件或黑洞文件中 

# 如果mysql是挂在docker里面的 需要指定IP和端口 本机也需要
innobackupex --user=root --password=123456 --host=10.0.6.55 --port=3303 ./db_backup/

# ==== nc方法 
# 流式备份，完成远程主机间的全量备份
# master端
innobackupex --user=root --password=123456 --host=10.0.6.55 --port=3303 --stream=tar ./ | nc 10.0.6.35 9998
# slave 端
nc -l 9998 | tar -ixvf -


# ==== 官方的xbstream
innobackupex  --user=root --password="upyun123" --port=3303 --host=10.0.6.55 --slave-info --parallel=16  --stream=xbstream ./ 2>/data/xtrabackup_log.txt | pv -q -L40m | ssh lily@172.26.1.245 " cat - |  xbstream -x -C /data/innobackup"


# 备份完成后会在/db_backup文件夹下面产生一个根据日期来创建的备份文件夹
# 自定义备份文件名
innobackupex --user=root --password=123456 --no-timestamp ./db_backup/test/ 2>>./db_backup/backup.log

# 需要清除数据，不然出现报错
systemctl stop mysqld
rm -rf /var/lib/mysql/*　　#危险操作，请在测试环境测试
innobackupex --apply-log ./db_backup/ --use-memory=16G  
chown -R mysql:mysql /var/lib/mysql　# 重新授权
systemctl start mysqld
```


#### 使用xtrabackup增量备份 
待填
https://www.centos.bz/2017/09/xtrabackup-backup-mysql-innodb/
cnblogs.com/zhoujinyi/p/4088866.html
https://www.cnblogs.com/gaogao67/p/10980642.html

#### xtrabackup 流式备份
如果需要将 机器上面的备份数据发送到远程机器，或者快速搭建主从，可以采用流式备份。
例如：这里将172.26.1.3的数据发送到 172.26.1.245 并做成主从同步。  master: 172.26.1.3  slave:172.26.1.245
172.26.1.3 备份之前需要先关闭多线程复制，否则会报错
```bash
mysql> show variables like '%worker%'
    -> ;
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| slave_parallel_workers | 0     |
+------------------------+-------+
1 row in set (0.00 sec)

mysql> set global slave_parallel_workers=0;
Query OK, 0 rows affected (0.00 sec)
```

```bash
# master端 
# 主要操作就是打包本地的数据，使用xbstream传输到对端机器，然后使用ssh登录对端 并且接收
innobackupex  --user=root --password="upyun123" --port=3303 --host=10.0.6.55 --slave-info --parallel=16  --stream=xbstream ./ 2>/data/xtrabackup_log.txt | pv -q -L40m | ssh lily@172.26.1.245 " cat - |  xbstream -x -C /data/innobackup"
```
注：这里的 172.26.1.245 是远程机器的ip

--slave-info： 备份目录下会多生成一个xtrabackup_slave_info文件，文件内容类似于:CHANGE MASTER TO MASTER_LOG_FILE='', MASTER_LOG_POS=0。 
    这个参数适用的场景：假设有主库A和从库B，目前想再添加一台备库C，并让备库C以主库A为master；因为主库A是生产库，压力一般比较大，所以就在备库B上备份一个数据库，然后把这个备份拿到C服务器上，并导入到C库，接下来再在C服务器上执行change master的命令：其中master_host是A的ip,而master_log_file和master_log_pos就是这个xtrabackup_slave_info里面的值。 

#### 备份恢复 
```bash
恢复（在远程机器172.26.1.245执行）
1，拷贝master机器的/etc/my.cnf 文件，并修改server_id。
2，将172.26.1.245做成172.26.1.3的从库， 从 /data/innobackup/xtrabackup_binlog_info 文件里面读取binlog和position信息。
#  cat /data/innobackup/xtrabackup_binlog_info 
mysql-bin.000066 94839444
mysql> CHANGE MASTER TO MASTER_HOST='172.26.1.3',MASTER_USER='repl',MASTER_PASSWORD='xxxxxx',MASTER_AUTO_POSITION=0,MASTER_LOG_FILE='mysql-bin.000066',MASTER_LOG_POS=94839444; 

mysql> start slave;
```

#### 区别
Xtrabackup备份时不能备份表结构、触发器等等，也不能智能区分.idb数据文件。另外innobackupex还不能完全支持增量备份，需要和xtrabackup结合起来实现全备的功能。



## xtrabackup详解
记录一下xtrabackup的备份流程
```bash
# 先备份
innobackupex --user=hzj --password=upyun123 --port=3303 --host=10.0.6.55 ./
cat /root/data/backup/2020-06-08_01-55-22/xtrabackup_checkpoints
# backup_type = full-backuped  # 这里提示backup 说明当前是备份的
# from_lsn = 0
# to_lsn = 6110542
# last_lsn = 6110542
# compact = 0
# recover_binlog_info = 0
# flushed_lsn = 12501828

# 回滚 
# 将全量备份后的内容拷贝到需要备份的从机器上进行备份
innobackupex --apply-log --use-memory=32M /root/data/backup/2020-06-08_01-55-22/
# 查看文件状态
more xtrabackup_checkpoints
# backup_type = full-prepared # 已经准备完成
# from_lsn = 0
# to_lsn = 2641979
# last_lsn = 2641988
# compact = 0
# recover_binlog_info = 0
# flushed_lsn = 12501828

# 备份
innobackupex --copy-back /root/data/backup/2020-06-08_01-55-22/
chown -R mysql.mysql /application/mysql/ # 修改权限
# 重启数据库
systemctl restart mysqld
```



## 错误

### docker mysql
```bash
docker run -p 3306:3306 --name mysql-master -v /opt/docker_v/mysql/conf:./master.cnf -e MYSQL_ROOT_PASSWORD=upyun123 -itd mysql:5.7
docker: Error response from daemon: invalid volume specification: '/opt/docker_v/mysql/conf:./master.cnf': invalid mount config for type "bind": invalid mount path: './master.cnf' mount path must be absolute.
```
这里挂在需要挂在绝对路径

### innobackupex 命令出错
```bash
[root@db01 mysql]# innobackupex --user=root --password=123456 --defaults-file=/application/mysql/my.cnf /data/mysql/backup/
xtrabackup: Error: --defaults-file must be specified first on the command line
[root@db01 mysql]# innobackupex --defaults-file=/application/mysql/my.cnf --user=root --password=123456 /data/mysql/backup/
190817 09:07:47 innobackupex: Starting the backup operation
```

需要把--defaults-file 作为第一个参数


### docker-mysql无法被innobackupex备份
使用过程中出现这个错误，这里做下记录
```bash
# 如果mysql是挂在docker里面的 需要指定IP和端口 本机也需要
innobackup --user=root --password=123456 --host=10.0.6.55 --port=3303 ./db_backup/
```

## docker-mysql重启数据库
因为mysql的配置文件会根据需求去更改，所以最好把配置文件从本地挂载进去
```bash
docker run -p 3306:3306 --name mysql-master -v /root/hzj/hzj/mysql/master.cnf:/etc/mysql/conf.d/master.cnf -e MYSQL_ROOT_PASSWORD=upyun123 -itd mysql:5.7

>c8328f40cfcb
```
修改配置文件重启数据库---> 重启docekr容器
```bash
docker restart c8328f40cfcb
```

## docker-mysql运行
```bash
# 挂在日志 数据和配置
docker run -p 3308:3306 --name mysql-slave \
-v /mydata/mysql-slave/log:/var/log/mysql \
-v /mydata/mysql-slave/data:/var/lib/mysql \
-v /mydata/mysql-slave/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7

```


## mysql fake data 

```bash

ALTER USER 'root'@'localhost' IDENTIFIED BY 'upyun123';
CREATE USER 'hzj'@'%' IDENTIFIED WITH mysql_native_password BY 'upyun123';
GRANT ALL PRIVILEGES ON *.* TO 'hzj'@'%';
create database datafordata  character set utf8mb4;
use datafordata;
CREATE TABLE `t_temp` (
	`id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
	`comment` varchar(10) NOT NULL DEFAULT '' COMMENT '备注',
	`create_time` bigint(20) NOT NULL ,
	`update_time` bigint(20) NOT NULL ,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
INSERT INTO `datafordata`.`t_temp` (`id`, `comment`, `create_time`, `update_time`) VALUES ('1', 'helloworld', '1', '1');
INSERT INTO `datafordata`.`t_temp` (`id`, `comment`, `create_time`, `update_time`) VALUES ('2', 'helloworld', '1', '1');
INSERT INTO `datafordata`.`t_temp` (`id`, `comment`, `create_time`, `update_time`) VALUES ('3', 'helloworld', '1', '1');
INSERT INTO `datafordata`.`t_temp` (`id`, `comment`, `create_time`, `update_time`) VALUES ('4', 'helloworld', '1', '1');
INSERT INTO `datafordata`.`t_temp` (`id`, `comment`, `create_time`, `update_time`) VALUES ('5', 'helloworld', '1', '1');
INSERT INTO `datafordata`.`t_temp` (`id`, `comment`, `create_time`, `update_time`) VALUES ('6', 'helloworld', '1', '1');
INSERT INTO `datafordata`.`t_temp` (`id`, `comment`, `create_time`, `update_time`) VALUES ('7', 'helloworld', '1', '1');
```

## extra
https://www.cnblogs.com/clsn/p/8138015.html#auto-id-40
https://segmentfault.com/a/1190000022440263
https://www.cnblogs.com/clsn/p/8150036.html#auto-id-0