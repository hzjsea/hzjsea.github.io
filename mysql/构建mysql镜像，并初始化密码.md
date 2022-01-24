## 构建镜像

> 镜像和容器

镜像是作为`模板` 容器是作为模板的不同`实例`

比如说我们可以定义模板可以定义所有的由该模板产生的mysql容器初始密码默认为`my-secret-pw` 就可以在dockerfile中这样写

```bash
FROM mysql:5.7
ENV MYSQL_ROOT_PASSWORD=my-secret-pw
```

如果不需要则直接使用`mysql:5.7` 在构建的时候用key的方式去指定密码

```bash
docker run container-id -d mysql:5.7
```

> 开始构建

构建mysql镜像，并初始化密码

```

FROM mysql:5.7
ENV MYSQL_ROOT_PASSWORD=my-secret-pw
```

`docker build . -t mysql2`

启动容器 并初始化配置

`挂载的时候，必须要绝对路径 不然会挂载不进去`

`docker run -p 12345:3306 -v /root/hzj/config/etc/:/etc/mysql/conf.d -d mysql2:lates`

`docker exec -it c00e931d9c01 /bin/bash`

![image-20210406110702071](/Users/alpaca/Library/Application Support/typora-user-images/image-20210406110702071.png)

> 报错

Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock'

这个问题 一直没有办法解决， 后面遇到在解决，大概是sock的权限不足 或是 不存在导致的

## mysql docker 搭建

mysql docker 需要挂载三个目录，不要一开始就去创建 ，`docker run`的时候会自动去创建

```bash
docker run -p 3306:3306 --name mysql \
 -v /mydata/mysql/log:/var/log/mysql \
 -v /mydata/mysql/data:/var/lib/mysql \
 -v /mydata/mysql/conf:/etc/mysql \
 -e MYSQL_ROOT_PASSWORD=root \
 -d mysql:5.7

 docker run -p 12347:3306 --name mysql \
 -v /root/yy/mydata/mysql/log:/var/log/mysql \
 -v /root/yy/mydata/mysql/data:/var/lib/mysql \
 -v /root/yy/mydata/mysql/conf:/etc/mysql \
 -d mysql2
```

参数说明

```
-p 3306:3306： 将容器的3306端口映射到宿主机的3306端口
--name mysql：将容器名字设置为mysql
-v /mydata/mysql/log:/var/log/mysql：将日志文件夹挂载到宿主机
-v /mydata/mysql/data:/var/lib/mysql：将数据文件夹挂载到宿主机
-v /mydata/mysql/conf:/etc/mysql：将配置文件夹挂载到宿主机
-e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码
-d mysql:5.7：后台运行容器，并返回容器ID
```

> docker-compose 启动

```dockerfile
version: '2'
services:
  mysql:
    container_name: mysql
    image: mysql:5.7
    ports:
      - "3306:3306"
    volumes:
      - /mydata/mysql/log:/var/log/mysql
      - /mydata/mysql/data:/var/lib/mysql
      - /mydata/mysql/conf:/etc/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - COMPOSE_PROJECT_NAME=mysql-server
```

> 查看挂载情况

```
docker inspect contariner-ID
```

> 挂载数据卷

```bash
[root@k8s-master01 ~]# cd /mydata/mysql/
[root@k8s-master01 mysql]# ls
conf  data  log
[root@k8s-master01 mysql]# cd conf/
[root@k8s-master01 conf]# pwd
/mydata/mysql/conf
```

> 创建mysql配置文件

```bash
cat > my.cnf <<-EOF
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
EOF
```

> 重启mysql 容器

```bash
docker restart mysql
```

> 进入容器查看配置

```bash
[root@k8s-master01 conf]# docker exec -it mysql /bin/bash
root@3873b46ee9b0:/# cd /etc/mysql/
root@3873b46ee9b0:/etc/mysql# ls
my.cnf
root@3873b46ee9b0:/etc/mysql# cat my.cnf 
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

> docker启动容器自启

```
docker update mysql --restart=always
```

## mysql 5.7 详细配置 kv

[MySQL 5.7 my.cnf配置文件说明-毛虫小臭臭-51CTO博客](https://blog.51cto.com/moerjinrong/2092791)




