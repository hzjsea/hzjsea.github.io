---
title: linux扩展软件命令
categories:
  - linux
abbrlink: 25384ae1
date: 2020-02-16 22:26:23
tags:
  - linux
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux扩展软件命令](#linux扩展软件命令)
  - [json工具](#json工具)
    - [json解析工具](#json解析工具)
      - [常用操作](#常用操作)
      - [格式化json](#格式化json)
      - [根据key值获取value值](#根据key值获取value值)
      - [多层key获取value值](#多层key获取value值)
      - [获取key值](#获取key值)
      - [判断是否存在某key](#判断是否存在某key)
      - [jq更高级的用法](#jq更高级的用法)

<!-- /code_chunk_output -->
<!-- more -->

# linux扩展软件命令

## json工具

### json解析工具
对于PHP和Python来说，解析json并不是太大的问题，同样的在linux下jq这个json解析工具也是相当完善的。
```bash
# 安装
sudo yum install jq   Centos安装
sudo apt-get install jq Ubuntu安装
# 使用
jq[参数列表] '过滤条件' 文件名或标准输入
jq --help 查看帮助文档
-compact-output / -c 默认情况下，jq会将json格式化为多树状结构输出，但有时需要将一个json串在一行输出，即可使用该参数
```
#### 常用操作
#### 格式化json
格式化json，方便查看,当然如果本身文件不是json的格式，他则会报错
```bash
cat xxx.json | jq . # 格式化
```
报错内容:
```bash
manu@manu:~/code/php/json$ cat json_err.txt |jq .
parse error: Expected separator between values at line 1, column 183
```
#### 根据key值获取value值
```bash
{
	"key_1":"value_1",
	"key_2":"value_2",
}
# 根据key值获取vlaue
cat som_json | jq '.foo'
```
如果值不存在，则返回null

#### 多层key获取value值
```bash
cat som_json | jq '.foo.bar'
cat som_json | jq '.foo[2].bar'
```

#### 获取key值
```bash
cat som_json | jq 'keys'
```

#### 判断是否存在某key
```bash
cat som_json | jq 'has("key")'

```

#### jq更高级的用法
http://justcode.ikeepstudying.com/2018/02/shell%EF%BC%9A%E6%97%A0%E6%AF%94%E5%BC%BA%E5%A4%A7%E7%9A%84shell%E4%B9%8Bjson%E8%A7%A3%E6%9E%90%E5%B7%A5%E5%85%B7jq-linux%E5%91%BD%E4%BB%A4%E8%A1%8C%E8%A7%A3%E6%9E%90json-jq%E8%A7%A3%E6%9E%90-json/

