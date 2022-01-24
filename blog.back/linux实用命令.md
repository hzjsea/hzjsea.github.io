---
title: linux实用命令
categories:
  - linux
tags:
  - linux
abbrlink: fa4e8e08
date: 2020-02-16 23:00:34
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux实用命令](#linux实用命令)
  - [查找包含某字符串的文件及位置](#查找包含某字符串的文件及位置)
  - [压缩打包与解压缩打包](#压缩打包与解压缩打包)
  - [软连接与硬连接](#软连接与硬连接)

<!-- /code_chunk_output -->
<!-- more -->


# linux实用命令

## 查找包含某字符串的文件及位置
```bash
find . | xargs grep -ri "要查找的字符串
```

## 压缩打包与解压缩打包
```bash
# 压缩
tar -jcv -f filename.tar.bz2 filename
tar -cxv -f filename.tar filename
# 查询压缩包中的内容
tar -jtv -f filename.tar.bz2 
# 解压
tar -cxv -f filename.tar .
tar -jxv -f filename.tar.bz2 -C 预解压缩的目录 
```

## 软连接与硬连接
在时间过程中发现一个问题
<!-- 现在我有一个文件夹 `test/index.md`  -->
然后我使用软连接到另外一个目录下面

```bash
ln -s ./test/ ./test1/
```

这时候test1下面也会有`index.md`，但是当我把这个test1下面的index.md删除的时候，test下面也会删除

另外一个场景
```bash
ln -s ./test/index.md ./test1/
```
这样创建则不会被删除