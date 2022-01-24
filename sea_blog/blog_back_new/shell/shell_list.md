---
title: shell实战
categories:
  - shell
tags:
  - shell
abbrlink: 6dab4d20
date: 2020-03-11 13:29:59
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [shell_list](#shell_list)
  - [服务器之间发包测试](#服务器之间发包测试)
  - [服务器ss健康状态检测](#服务器ss健康状态检测)
  - [查询inventoy](#查询inventoy)
  - [处理Inventory上的内容](#处理inventory上的内容)
  - [搭建使用supervisor管理的redis单机实例](#搭建使用supervisor管理的redis单机实例)
  - [交换机生成聚合口脚本](#交换机生成聚合口脚本)
  - [shell获取当前文件目录下面的所有文件](#shell获取当前文件目录下面的所有文件)

<!-- /code_chunk_output -->


<!-- more -->


# shell_list

## 服务器之间发包测试
```bash
#!/bin/bash

set -ex

# more_info=""
more_info="-v"
os_version="cat /etc/redhat-release"
vip="10.0.6.55"
vport="8081"

record="/tmp/record.txt"

_clean() {
	rm -rf ${record}
}

_server() {
	# nc -v -e /bin/bash -ltkp 8081
	if [ ${os_version} == "CentOS release 6.5 (Final)" ]; then
		nc ${more_info} -e /bin/bash -ltk ${vport}
	else
		nc ${more_info} -e /bin/bash -ltkp ${vport}
	fi

}

_client() {
	for i in {1..1000}; do
		echo "=================${i}==================="
		# nc -v 10.0.6.55 8081
		echo "echo 1 >> ${record}" | nc ${more_info} ${vip} ${vport}
	done
	echo "cat ${record} | wc -l >> ${record}" | nc ${more_info} ${vip} ${vport}
}

main() {

	case $1 in
	server)
		_server
		;;
	client)
		_client
		;;
	clean)
		_clean
		;;
	default)
		echo "1"
		;;
	*)
		echo "1"
		;;
	esac
}

main $@

```

## 服务器ss健康状态检测
```shell
#!/bin/sh
state="established"
timestamp=`date "+%d/%b/%Y:%X"`
tmpfile="z"
hostname=$(hostname)

echo "'"  > $tmpfile.json
ss2 -4itn state $state dport = :80 or sport = :80 or dport = :443 or sport = :433 or sport = :8100 or dport = :8100 |
awk '{ if(NR > 1 && $3 !~ /127.0.0.1/ && $3 !~ /rt[to]/ ) {
		line=$0; getline; if($0~/minrtt/) print line""$0
  	}
}' |
awk '{
	split($3,src,":"); Saddress=$3;
	split($4,dst,":"); Daddress=dst[1];Dport=dst[2];
	node_type="network-attack";
        split($8,a,"[:/]");
        split($13,b,"[:/]");
        if ($0 ~ /retrans:/){
                split($0,c,"retrans:");
                gsub("/.*","",c[2]);
                Retrans=c[2]
        }else{
                Retrans=0
        };
        if ($0 ~ /send/){
                split($0,d,"send");
                gsub("bps.*","",d[2]);
                if(d[2] ~ /K/) {
                        gsub("K","",d[2]);
                        Send=d[2]/1000
                }
                else if(d[2] ~ /M/) {
                        gsub("M","",d[2]);
                        Send=d[2]
                }
                else {
                        Send=d[2]
                }
        }; 
        print(Daddress,Saddress,a[2],b[2],Retrans,Send)
}' |
awk '{
    a[$2]++;d[$2]=$1;rtt[$2]+=$3;cwnd[$2]+=$4;retrans[$2]+=$5;send[$2]+=$6
    }END{
        for (i in a){ 
            z=split(i,q,":")
            nums=0
            if (retrans[i]/a[i] != 0) {
                    nums=int(retrans[i]/a[i]+1)
                    }
            else{
                nums=retrans[i]/a[i]
          }
            printf("{\
\"Daddress\":\"%s\", \
\"Saddress\": \"%s\",\
\"SPort\": \"%s\",\
\"rtt\": %d,\
\"cwnd\": %d,\
\"Retrans\": %d,\
\"send\": %d,\
\"total\": %d,\
\"line_state\": \"%s\",\
\"type\": \"%s\",\
\"node_type\": \"%s\",\
\"timestamp\": \"%s +0800\",\
\"hostname\": \"%s\"\
}\n",
                    d[i],q[1],q[2],rtt[i]/a[i],cwnd[i]/a[i],nums,send[i]/a[i],a[i],"ESTAB","bbr","network-attack","'$timestamp'","'$hostname'")}
}' >> $tmpfile.json

echo "'" >> $tmpfile.json

curl -XPOST http://chopper.x.upyun.com:4000/v2/log/repo/cdn-common  -H 'Apikey:1ndT12Jj85UsXQdat2KOMBEiiT28rsIQ' -H "Content-Type:application/json" -T $tmpfile.json -v -D-

```
ss 各个字段检测
https://kkc.github.io/2017/12/18/Network-Performance-Tuning/



## 查询inventoy
查询inventoy 下面不是CWS的机器  
```shell
#!/usr/bin/bash
grep "LNA" mingtao/inventory/machines/lists-* | awk -v FS='[:= ]'  '{ if ($2 ~/CWS/) {print "1"} else { print $2 ,$4 }}'    > ./hezejun/xx
```


## 处理Inventory上的内容
处理表格生成json格式的数据
```bash
#!/usr/bin/env bash


# 根据inventory上的文件生成json
# 格式如下
# CTN-ZJ-LNA-S02 ansible_ssh_host=183.134.101.131 uplink=T1/1/1
# ZEN-CN-HKG-S01 ansible_ssh_host=129.227.137.225 uplink=T1/0/21 T1/0/22
# CMN-SC-CTU4-S01 ansible_ssh_host=117.172.22.129 uplink=T1/0/47 T1/0/48
# CUN-FJ-XMN-S01 ansible_ssh_host=36.248.208.226 uplink=T2/0/49
# CUN-SD-DEZ-S01 ansible_ssh_host=61.179.240.158 uplink=T1/0/49
# CTN-SN-XIY-S01 ansible_ssh_host=1.81.5.161 uplink=B100

# target_file="/Users/alpaca/PycharmProjects/Npc/switch/lists-switch-edge"
target_file="/Users/alpaca/PycharmProjects/Npc/switch/lists-switch"

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
# cat $target_file

```


## 搭建使用supervisor管理的redis单机实例
```bash

#!/bin/bash

_init() {
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

# 准备环境
_install() {
    mkdir -p /root/package && mkdir -p /usr/local/redis
    mkdir -p /var/log/redislog
    yum install wget epel-release -y
    yum install gcc bc bash-completion automake autoconf libtool make git -y
    wget http://download.redis.io/releases/redis-5.0.8.tar.gz -O /root/package/redis.tar.gz
    tar zxvf /root/package/redis.tar.gz -C /usr/local/redis --strip-components 1
    cd /usr/local/redis/ && make distclean && make && make install

    ln -fs /usr/local/redis/src/redis-server /usr/local/bin/redis-server
    ln -fs /usr/local/redis/src/redis-cli /usr/local/bin/redis-cli
    ln -fs /usr/local/redis/src/redis-sentinel /usr/local/bin/redis-sentinel
}

# 准备supervisor
_supervisor() {
    [ -d chenyuecc ] && (rm -r chenyuecc)
    git clone -b supervisor https://gitee.com/Alpaca-H/chenyuecc.git && cd /root/chenyuecc/ && sh ./run.sh init 2>/dev/null
}

# 准备redis
_redis() {
    [ ! -d /usr/local/redis/ ] && (
        echo "please install redis"
        exit 0
    )
    [ -d chenyuecc ] && (rm -r chenyuecc)
    git clone -b redis https://gitee.com/Alpaca-H/chenyuecc.git && cp /root/chenyuecc/redisCC/* /usr/local/redis/
}

# 准备单机redis
_redis_single() {
    cat >/etc/supervisor.d/redisgo.conf <<EOF
[program:redis]
command=/usr/local/redis/src/redis-server  /usr/local/redis/redis.conf
autostart=true
process_name=redis
autostart=true
autorestart=true
startretries=10
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

_down() {
    echo "关闭"
}



main() {

    [ ! -z $frontcolor ] || _init
    [ -d "/root/package" ] || _install
    action=${1:-"default"}

    case ${action} in
    redis | supervisor )
        _${action}
        ;;
    redisgo)
        _redis
        _supervisor
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

## 交换机生成聚合口脚本
```bash
#!/bin/bash

# member id
portType="1"

# speed
portName="Ten"
# portName="G"


# bridgeNumber

start_port=1
end_port=24


for i in $(seq $start_port $end_port);do
    echo "interface bridge-aggregation ${i}"
    echo "interface ${portName}${portType}/0/$(($i*2-1))"
    echo "port link-aggregation group ${i}"
    echo "interface ${portName}${portType}/0/$(($i*2))"
    echo "port link-aggregation group ${i}"
done
```


## shell获取当前文件目录下面的所有文件
```
# 文件格式如下
-rw-r--r-- 1 root root 780M Jan 13 00:52 _00000_20210109012317
-rw-r--r-- 1 root root 844M Jan 13 00:52 _00001_20210109012417
-rw-r--r-- 1 root root 852M Jan 13 00:52 _00002_20210109012517
-rw-r--r-- 1 root root 851M Jan 13 00:52 _00003_20210109012617
-rw-r--r-- 1 root root 832M Jan 13 00:52 _00004_20210109012717
-rw-r--r-- 1 root root 782M Jan 13 00:52 _00005_20210109012817
-rw-r--r-- 1 root root 797M Jan 13 00:52 _00006_20210109012917
-rw-r--r-- 1 root root 708M Jan 13 00:52 _00007_20210109013017
-rw-r--r-- 1 root root 722M Jan 13 00:52 _00008_20210109013117
-rw-r--r-- 1 root root 680M Jan 13 00:52 _00009_20210109013217
-rw-r--r-- 1 root root  174 Jan 13 01:05 xx.sh
```

```bash
find . -type f -size +500M  | cut -c 3- | while read line;do
        tshark -r $line  -Y 'http.request.uri matches "1eecb132-e1ff-406e-9788-5b92339284f8"' > ${num:1:5}.log
done
```