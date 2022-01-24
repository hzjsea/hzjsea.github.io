---
title: vim快捷键
categories:
  - vim
tags: 
  - vim
abbrlink: e47927c
date: 2020-01-09 11:08:30
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [vim快捷键](#vim快捷键)
  - [快捷键](#快捷键)
  - [编辑](#编辑)
  - [删除特定行数之间的空行](#删除特定行数之间的空行)
  - [多行递进](#多行递进)

<!-- /code_chunk_output -->


<!-- more -->
# vim快捷键
VIM 是 Linux 系统上一款文本编辑器，它是操作 Linux 的一款利器。
当前有很多优秀的 IDE 都支持安装 VIM 插件，原因就是使用它便捷，高效，很爽！
主要记录了 VIM 的一些常用使用技巧，方便随时查阅学习 。

## 快捷键
```bash
# 跳转到行首  shift+i 
# 跳转到行尾 esc->shift+a
```

## 编辑
- 清空当前行，并进入编辑模式 cc
- 清空当前单词，并进入编辑模式 cw


## 删除特定行数之间的空行
```bash
:68,121g/^s*$/d  # 这里^s*$ 匹配所有的空行
```

## 多行递进
```bash
可视模式v+方向键 选中要递进的内容
shift + <  > 两个键位空值递进方向 一次表示一个空格

```