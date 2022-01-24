# linux中编译用到的命令
./configure  、 make 、make install 

1. 其中./configure是用来检测你的安装平台的目标特征的。比如它会检测你是不是有CC或GCC，并不是需要CC或GCC，它是个shell脚本。
2. make是用来编译的，他从makefile中读取指令，然后进行编译
3. make install 是用来安装的，他从makefile中读取指令，安装到指定的位置

在lfs搭建过程中，我们要经常使用到这些命令


## configure

### 如何生成configure的文件
 https://www.cnblogs.com/bugutian/p/5560548.html

### 
比如我们会进入gcc的源码文件中执行configure生成makefile文件
比如:
```bash
../gcc-4.9.2/configure                             \
    --target=$LFS_TGT                              \
    --prefix=/tools                                \
    --with-sysroot=$LFS                            \
    --with-newlib                                  \
    --without-headers                              \
    --with-local-prefix=/tools                     \
    --with-native-system-header-dir=/tools/include \
    --disable-nls                                  \
    --disable-shared                               \
    --disable-multilib                             \
    --disable-decimal-float                        \
    --disable-threads                              \
    --disable-libatomic                            \
    --disable-libgomp                              \
    --disable-libitm                               \
    --disable-libquadmath                          \
    --disable-libsanitizer                         \
    --disable-libssp                               \
    --disable-libvtv                               \
    --disable-libcilkrts                           \
    --disable-libstdc++-v3                         \
    --enable-languages=c,c++
```
其中prefix=/tools 表示他将目标软件安装在/tools下

## make
生成完makefile文件后，就交给make来进行编译

-j<number> 指定多进程
查看自己的机有几个内核  more /proc/cpuinfo |grep "physical id"|uniq|wc -l器

### 自己编写makefile文件



```bash 
# 如果makefile中的前置条件不存在的话，就会报错
# xxx.txt
a.txt: b.txt c.txt
    cat b.txt c.txt > a.txt

make --file=xxx.txt
```

http://www.ruanyifeng.com/blog/2015/02/make.html
https://www.cnblogs.com/bugutian/p/5560548.htm




### make install
这条命令来进行安装（当然有些软件需要先运行 make check 或 make test 来进行一些测试），这一步一般需要你有 root 权限（因为要向系统写入文件）。

### make clean
删除临时文件

### make distclean 
除了清除可执行文件和目标文件外，把configure所产生的Makefile也清除掉。