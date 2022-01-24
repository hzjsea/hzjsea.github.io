### tree目录树的参数
```text
-a 显示所有文件和目录
-C 在文件和目录清单上添加色彩，便于区分各种类型
-d 显示称而非内容
-D 列出文件或目录的更改时间
-s 列出文件或目录大小
-t 用文件和目录的更改时间排序
-u 列出文件或目录的拥有者名称
```
更多参数:https://www.runoob.com/linux/linux-comm-tree.html

### Linux的目录结构
**管理类**
- /   根目录
- /bin 所有用户可用的基本命令文件 在centos7之后 /bin 软连接到 /usr/bin 也就是说/bin是/usr/bin的快捷方式
- /sbin 供系统管理使用的工具程序,超级权限用户root的可执行命令存放   普通用户无法执行该目录下的文件   /sbin 软连接到/usr/sbin
- /var 这个目录是经常变动的，用来存储经常被修改的文件，如日志，邮箱等
- /etc 主要存放系统配置方面的文件
- /dev 存储特殊文件或设备文件

**用户类**
- /home 主要存放个人数据
- /root 系统管理员用户

**应用程序类目录**
- /tmp 为那些会产生临时文件的程序提供的用户存储临时文件的目录 
- /lib 该目录用来存放系统动态链接共享库，几乎所有的应用程序都会用到该目录下的共享库
- /usr 存放一些不适合放在/bin 或/etc下的额外工具，如个人安装的程序或工具
- /usr/local 本地化配置文件
- /usr/bin  主要存放程序
- /usr/share 主要存放一些共享数据
- /usr/lib 存放一些不能直接运行，但却是许多程序运行所必需的一些函数库文件

**其他重要目录**
- /etc/rc.d 放置开关机脚本
- /etc/rc.d/init.d 放置启动脚本
- /usr/share/doc 系统说明文件的放置目录
- /usr/share/man 程序说明文件放置目录
- /usr/src 内核源代码目录


- /boot 引导加载器必须用到的各静态文件
- /home 普通用户的家目录的集中位置
- /root
- /lib64 64位系统特有的存放64位共享库的路径   /lib64 软连接到 /usr/lib64
- /media 其他文件系统的临时挂载点
- /opt 附加应用程序的安装位置，可选路径
- /srv 当前主机为服务提供的数据


**其他重要目录**
/etc/resolv.conf 该文件是由域名解析器(resolv,一个根据主机名解析IP地址的库)使用的配置文件

<summary>配置摘要</summary>
```
配置格式:
关键字开头 + 多个空格+ 参数
关键字:
nameserver //定义DNS服务器的IP地址
domain //定义本地域名
search //定义域名的搜索列表
sortlist //对返回的域名进行排序
```

/etc/local 该文件主要是本地化配置文件

<summary>配置摘要</summary>
```
完全支持中文环境,但是以英文作为用户界面
LANG=zh_CN.utf8
LC_MESSAGES=en_US.utf8
其余参数查询
http://man7.org/linux/man-pages/man7/locale.7.html
```


Centos7目录详细:https://meetes.top/2018/08/05/CentOS7%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84%E8%AF%A6%E7%BB%86%E7%89%88/


## 文件查看命令
cat 将文件内容打印到屏幕
```bash
cat > xxx.md <<EOF
>test
>EOF 
新建或覆盖文件

cat >> xxx.md <<EOF
>test
>EOF
追加写入内容
```
more 分页浏览 --- 回车下一行  空格翻页
less 分页浏览 --- 可以反复浏览  q退出  上下左右控制
head 从文件头部开始看  默认10行   -n num  指定输出行数
tail 从文件尾部开始看  默认10行   -n num  指定输出行数   -f filename 跟踪文件(及文件更新的时候命令行会热更新动态)






### 文件查找命令
1. locate 
locate 查找文件或目录 他会建立一个系统内所有档案名称以路径的数据库，之后在查找的过程中，则会进入数据库中查找，而不会深入档案系统之中查找。更新数据库使用updatedb


安装locate  yum install mlocate -y
出现错误:locate: can not stat () `/var/lib/mlocate/mlocate.db': No such file or directory
解决方法: updatedb

查找 pwd的文件  locate pwd

2. which 查找系统PATH变量目录下的命令(绝对路径)
3. whereis 查找文件索引数据库下的命令、源文件、man文件、非PATH文件等z