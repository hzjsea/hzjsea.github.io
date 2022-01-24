## 构建临时系统
在(1)中说过我们所编译和临时系统的构建都是在我们所挂载的文件上执行的，并且我们还做了用户的隔离防止破坏宿主系统的环境

首先确保一下我们的当前目录是在/opt/lfs/sources下的，以及当前的用户是lfs
```bash
pwd
/opt/lfs/sources
whoami 
lfs
```

文件的路径很重要，错了就可能在编译的时候出现各种错误

### 编译Binutils-2.25包
```bash
cd $LFS/sources
tar -xf binutils-2.25.tar.bz2 
cd binutils-2.25
mkdir -v  $LFS/sources/binutils-build
cd /opt/lfs/sources/binutils-build

# 编译参数  生成makefile文件
../binutils-2.25/configure \
 --prefix=/tools \
--with-sysroot=$LFS \
 --with-lib-path=/tools/lib \
 --target=$LFS_TGT \
 --disable-nls \
 --disable-werror

# 开始编译
make

# 构建符号链接
case $(uname -m) in
 x86_64) mkdir -v /tools/lib && ln -sv lib /tools/lib64 ;;
esac

# 安装
make install
```

### 编译gcc包
gcc包很难编译，如果错误了 可以删除了重来，回退到binutils刚编译完成之后的样子就可以了

其实每一个包的编译流程都类似。只不过不同的包可能会有不同的设置。
比如这里的gcc包，首先我们要解压gcc的压缩包，进入后将mpfr gmp mpc 解压到里面，在gcc之后的编译会用到他们
```bash
tar -xf gcc-4.9.2.tar.bz2
cd gcc-4.9.2
tar -xf ../mpfr-3.1.2.tar.xz
mv -v mpfr-3.1.2 mpfr
tar -xf ../gmp-6.0.0a.tar.xz
mv -v gmp-6.0.0 gmp
tar -xf ../mpc-1.0.2.tar.gz
mv -v mpc-1.0.2 mpc

# 修改gcc的动态连接器安装位置 为/tools
for file in \
 $(find gcc/config -name linux64.h -o -name linux.h -o -name sysv4.h)
do
 cp -uv $file{,.orig}
 sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' \
 -e 's@/usr@/tools@g' $file.orig > $file
 echo '
#undef STANDARD_STARTFILE_PREFIX_1
#undef STANDARD_STARTFILE_PREFIX_2
#define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
#define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
 touch $file.orig
done
# 修复gcc的栈保护
sed -i '/k prot/agcc_cv_libc_provides_ssp=yes' gcc/configure
# gcc的编译最好在gcc-4.9.2这个包外面
mkdir -v ../gcc-build
cd ../gcc-build

# 生成makefile文件
../gcc-4.9.2/configure \
 --target=$LFS_TGT \
 --prefix=/tools \
 --with-sysroot=$LFS \
 --with-newlib \
 --without-headers \
 --with-local-prefix=/tools \
 --with-native-system-header-dir=/tools/include \
 --disable-nls \
 --disable-shared \
 --disable-multilib \
 --disable-decimal-float \
 --disable-threads \
 --disable-libatomic \
 --disable-libgomp \
 --disable-libitm \
 --disable-libquadmath \
 --disable-libsanitizer \
 --disable-libssp \
 --disable-libvtv \
 --disable-libcilkrts \
 --disable-libstdc++-v3 \
 --enable-languages=c,c++

 # 编译并安装
 make -j8 && make install
# 如果你出现了错误 如
Gcc compilation “cannot compute suffix of object files: cannot compile”
checking for suffix of object files... configure: error: in `/mnt/lfs/sources/gcc-build/x86_64-lfs-linux-gnu/libgcc'

https://www.linuxquestions.org/questions/linux-from-scratch-13/%5Bsolved%5D-error-compiling-gcc-in-lfs-4175558261/

# 建议重新编译或者删除重新再来
```



<font color='red'>除个别情况外，gcc安装完成之后，后面的安装就会很简单了</font> 按照教程，分别安装第一次临时系统所需要的软件
https://lctt.github.io/LFS-BOOK/lfs-systemd/index.html



## 错误合集
```bash 
checking for C compiler default output file name... a.out
checking whether the C compiler works... configure: error: in `/mnt/sources/binutils-build':
configure: error: cannot run C compiled programs.
If you meant to cross compile, use `--host'.
See `config.log' for more details.
```
可能是glibc构建错误的问题，重新编译glibc以及binutils-build


//ncurses
** Configuration summary for NCURSES 5.9 20110404:

     extended funcs: yes
     xterm terminfo: xterm-new

      bin directory: /tools/bin
      lib directory: /tools/lib
  include directory: /tools/include
      man directory: /tools/man
 terminfo directory: /tools/share/terminfo