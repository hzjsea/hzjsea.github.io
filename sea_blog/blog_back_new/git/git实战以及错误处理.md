---
title: git实战以及错误处理
categories:
  - 技术补充
tags:
  - git
abbrlink: 6db7a7cb
date: 2020-04-17 12:05:21
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [git实战以及错误处理](#git实战以及错误处理)
  - [应用篇](#应用篇)
    - [从远程仓库获取最新代码合并到本地分支](#从远程仓库获取最新代码合并到本地分支)
    - [git 上传忽略文件](#git-上传忽略文件)
  - [错误篇](#错误篇)
    - [git pull的时候出现错误](#git-pull的时候出现错误)
    - [gitlab 分支保护的问题](#gitlab-分支保护的问题)
    - [git pull拉取出错](#git-pull拉取出错)

<!-- /code_chunk_output -->

<!-- more -->

# git实战以及错误处理


## 应用篇

### 从远程仓库获取最新代码合并到本地分支
```bash
git remote -v # 查看当前远程的版本
git pull origin master # 拉取远端origin/master分支并合并到当前分支
git pull origin dev # 拉取远端origin/dev分支并合并到当前分支
```

<font color='red'>因为是直接合并，无法提前处理冲突。 不推荐</font>

<font color='red'>推荐方式1 获取最新代码到本地，然后手动合并分支</font>

```bash
# 查看当前远程的版本
$ git remote -v 
#获取最新代码到本地临时分支(本地当前分支为[branch]，获取的远端的分支为[origin/branch])
$ git fetch origin master:master1  [示例1：在本地建立master1分支，并下载远端的origin/master分支到master1分支中]
$ git fetch origin dev:dev1[示例1：在本地建立dev1分支，并下载远端的origin/dev分支到dev1分支中]
# 查看版本差异
$ git diff master1 [示例1：查看本地master1分支与当前分支的版本差异]
$ git diff dev1    [示例2：查看本地dev1分支与当前分支的版本差异]
# 合并最新分支到本地分支
$ git merge master1    [示例1：合并本地分支master1到当前分支]
$ git merge dev1   [示例2：合并本地分支dev1到当前分支]
# 删除本地临时分支
$ git branch -D master1    [示例1：删除本地分支master1]
$ git branch -D dev1 [示例1：删除本地分支dev1]
```

<font color='red'>推荐方式2 不额外建立本地分支</font>

```bash
# //查询当前远程的版本
$ git remote -v
# //获取最新代码到本地(本地当前分支为[branch]，获取的远端的分支为[origin/branch])
$ git fetch origin master  [示例1：获取远端的origin/master分支]
$ git fetch origin dev [示例2：获取远端的origin/dev分支]
# //查看版本差异
$ git log -p master..origin/master [示例1：查看本地master与远端origin/master的版本差异]
$ git log -p dev..origin/dev   [示例2：查看本地dev与远端origin/dev的版本差异]
# //合并最新代码到本地分支
$ git merge origin/master  [示例1：合并远端分支origin/master到当前分支]
$ git merge origin/dev [示例2：合并远端分支origin/dev到当前分支]
```


### git 上传忽略文件
每次本地写完项目，上传文件都时候，都会上传一些预编译的文件，比如python中的`__pycache__`文件，这个时候我们就要用.gitignore文件来定义忽视的文件
github上有一个开源项目写了很多程序的忽略文件，可以参考一下
https://github.com/github/gitignore
<font color='red'>  另外这个.gitignore文件只能在首次上传的时候有用，如果是已经存在于仓库上的，那就回出现错误</font>
`.gitignore`文件与`.git`文件同目录，上传的时候查看

```bash
# 未上传的时候 查看git add可以添加哪些文件
git status
# 添加到缓存后，查看暂存中有哪些文件
git status
# 重置add操作
git reset
```



## 错误篇

### git pull的时候出现错误
```bash
error: Your local changes to the following files would be overwritten by merge:
	xxx/xxx/xxx.java
Please, commit your changes or stash them before you can merge.
Aborting
```
意说是在你更新代码之前，你已经有了和远程仓库上有区别的代码块，可能是你修改了文件，但是远程仓库上没有修改
这个时候就要看自己的需求了
- 如果符合自己的本地的 直接commit 
- 如果是符合远程仓库的，就先`git stash`


### gitlab 分支保护的问题
![2020-07-08-10-08-50](http://noback.upyun.com/2020-07-08-10-08-50.png)
```bash
 GitLab: You are not allowed to force push code to a protected branch on this project.
```
![2020-07-08-10-12-37](http://noback.upyun.com/2020-07-08-10-12-37.png)
这是因为gitlab项目默认禁止push的
现在有两个解决方法 一种是先clone下来，就可以正常push了
另外一种是解除当前push用户的push权限
![2020-07-08-10-14-41](http://noback.upyun.com/2020-07-08-10-14-41.png)


### git pull拉取出错
git pull的时候发生冲突的解决方法之`error: Your local changes to the following files would be overwritten by mergea`
这里是出于一种对本地文件的的保护，本身项目分支分为master和manager两个分支，如果直接修改了master里面的分支
再去拉取master的远程分支的话，就会出现这个情况。

解决办法1:  如果你要保证线上的项目，而不是本地的项目更新到线上，这个方式不可取
- 保存当前工作区的内容，，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。
- 提交项目
- 从git栈中读取最近一次保存的内容，恢复工作区的相关内容，这是一个栈式的存储，pop会从最近的一个stash中读取内容并恢复
```bash
git stash
git commit
git stash pop
```

解决方法2: 保证线上的项目下拉，本地的项目被更新
- 放弃本地需改，直接覆盖
```bash
git reset --hard
git pull
```

解决方法3: 
如果修改的内容比较少，就可以直接重构项目重新修改
删除当前项目，重新拉取远程包
