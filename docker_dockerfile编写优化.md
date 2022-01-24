---
title: dockerfile编写优化
date: 2021-05-21 10:48:44
categories:  [docker]
tags: [docker]
---


<!--more-->


# dockerfile编写优化



1. 小镜像

要想镜像小，其它都不考虑可以用 scratch (无sh)，不过我们线上都是用的 alpine，承载着海量日活没有问题
每个镜像基本上都有alpine版本的镜像

其实实用性上ubuntu更好用些，当然如果是紧致要求的话，就可以试着用alpine和 scratch

2. 体积小

测试的时候run可以分开写，这样有利于调试，rebuild的时候，会在断层处直接开始编译，
但是考虑到体积的问题，run在正式上线的时候，最好还是合并一下

Dockerfile 最佳实践中尽量减少层，比如将多个 RUN 合成一个，例如
RUN apk update --no-cache
RUN apk add --no-cache ca-certificates
RUN apk add --no-cache tzdata
可以写成
RUN apk update --no-cache && apk add --no-cache ca-certificates tzdata

3. 设置镜像时区

RUN apk add --no-cache tzdata
ENV TZ Asia/Shanghai

4. 多利用标签

引用基础镜像的时候明确指明标签，不需要docker的daemon去猜测是什么版本，默认会去装最新的

打包自己的dockerfile的时候，多利用标签，
`docker build -t=“ centos:wordpress" .`
另外`:`后面写明镜像的作用
![](https://noback.upyun.com/2021-05-21-11-09-13.png!)


5. 充分利用镜像缓存

这个也是debug镜像时候提高效率的方法
由 Dockerfile 最终构建出来的镜像是在基础镜像之上一层层叠加而得，因此在过程中会产生一个个新的 镜像层。Docker daemon 在构建镜像的过程中会缓存一系列中间镜像。
docker build 镜像时，会顺序执行 Dockerfile 中的指令，并同时比较当前指令和其基础镜像的所有子镜像，若发现有一个子镜像也是由相同的指令生成，则 `命中缓存`，同时可以直接使用该子镜像而避免再去重新生成了。
为了有效地使用缓存，需要保证 Dockerfile 中指令的 连续一致，尽量将相同指令的部分放在前面，而将有差异性的指令放在后面,这里说的相同指令指的是，不容易出错的指令，比如一些固定文件，固定需要安装的软件，都可以放在前面

之前有打包过一些内容，需要在镜像中`apt install `一些软件
```bash
apt install xx yy zz 
```
下了半天，然后后面我发现少装了一些软件，然后去加了一个，于是缓存就失效了，打包的时候需要重新下载所有的包


6. add 与 copy volumes
这个很简单，固定住就好了，
- 需要复制解压用copy
- 需要数据持久化用volumes
- 其他全用copy


7. CMD 和 ENTRYPOINT 指令 的正确理解使用
他们两个的区别是

ENTRYPOINT 指令部分固定不变，容器运行时是无法修改的
而 CMD 部分的指令也可以改变，表现在运行容器时，docker run 命令中提供的参数会覆盖 CMD 的指令内容

因此可以利用他们的特性， 
- ENTRYPOINT作为容器启动的脚本，
- cmd作为可变部分参数

比如要运行容器执行 `ls -al`

```dockerfile
FROM debian:latest

MAINTAINER codesheep@163.com

ENTRYPOINT [ "ls", "-l"]   // 固定执行 ls -l
CMD ["-a"] // 参数 -a
```

然后运行的时候你想执行 `ls -latr`

```bash
docker build . -t debin:0.1 
docker run -it --name debin_test debin:0.1 -atr   // ls -altr
docker run -it --name debin_test2 debin:0.1 -a    // ls -al
```

8. 端口映射不要写死
尽量在docker容器外使用expose
```bash
＃尽量避免这种方式
EXPOSE 8080:8899

＃选择仅仅暴露端口即可，端口映射的任务交给 docker run 去做
EXPOSE 8080

docker run -itd -p 8881:8080 debian:0.4 
左边宿主，右边容器
```
![](https://noback.upyun.com/2021-05-21-11-32-34.png!)


## dockerfile 就那么几个指令

```bash
FROM  镜像源
MAINTAINER hzjsea
ENV TZ Asia/Shanghai

RUN echo xxx && echo xxx \
    RUN echo xxx
RUN xx.sh

ADD  xx.tar xx/
COPY file1 to_file
WORKDIR
CMD
ENTRYPOINT
CMD
```


https://blog.fundebug.com/2017/05/15/write-excellent-dockerfile/