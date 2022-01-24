---
title: git基础
date: 2020-12-02 13:39:14
categories: [技术补充,]  
tags: [git]
---


<!-- more -->

# git基础


声明几个目录,方便看懂文章
```bash
工作目录 - 初始化了git的目录
跟踪文件(暂存区文件) - git add README.md 表示将README纳入版本控制,表示已经被跟踪
暂存区快照 -- git commit -am "update" 后的文件
远程仓库 -- git push 上传远程仓库的内容
```

## 命令
```bash
## 全局设置用户信息
git config --global user.name "your name"
git config --global user.email "xxx@xxx.com"

## 设置当前仓库用户信息
git config user.name "your name"
git config user.email "xxx@xxx.com"

## 初始化文件
git init .

## 跟踪文件
## 将文件放到暂存区当中
git add <file>

## 查看跟踪文件的状态信息
git status 

## 简短的查看文件的状态信息
git status -s 
git status --short

## 查看同一文件暂存与未暂存的对比
git diff <file>
git diff

## 查看已经暂存的变化
git diff --cache

## 提交更新 ,到暂存区快照
git commit -m "update"

## 放入暂存区并提交更新, 也就是跳过了暂存区的步骤
git commit -am "update"

## 保留本地文件,删除暂存区快照中的文件
git rm <file> --cached

## 删除本地文件以及暂存区快照中的文件
git rm <file>

## 查看分支 
git remote -v

## 切换并创建分支
git checkout -b backup
```


## git使用

### 上传到暂存区
工作目录下面的文件,不外乎两种状态`已跟踪` 和 `未跟踪`
已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后， 它们的状态可能是未修改，已修改或已放入暂存区
纳入版本控制的文件的状态
![2020-12-02-13-46-11](http://noback.upyun.com/2020-12-02-13-46-11.png)

我们添加一个文件,现在这个文件是未跟踪的,文件状态是`untracked`
![2020-12-02-13-48-37](http://noback.upyun.com/2020-12-02-13-48-37.png)
接下来我们跟踪这个文件,文件的状态是`Changes to be committed:`,这个状态表示文件在暂存区当中
![2020-12-02-13-50-15](http://noback.upyun.com/2020-12-02-13-50-15.png)
如果我们这个时候再去修改,文件的状态是`Changes not staged for commit`,这个表示文件不在暂存区当中,我们需要使用`git add`将他添加到暂存区当中
另外这里我们还能看到,`README`文件存在两种状态,一个是在暂存区中,一个不在暂存区中,这是因为只有一部分内容是在暂存区当中的,修改的部分不存在,如果要合并到赞存档中,只要重新`git add README`就可以了
![2020-12-02-13-54-09](http://noback.upyun.com/2020-12-02-13-54-09.png)
![2020-12-02-13-55-23](http://noback.upyun.com/2020-12-02-13-55-23.png)

### 查看本地仓库状态
使用`git status -s `或者`git status --short` 简短的查看文件状态
状态信息如下
- ?? 表示没有被追踪的文件
- M 表示不在暂存区,但是被修改了的
- AM 表示被添加到暂存区后,又被修改了
- A 表示在暂存区当中的
![2020-12-02-14-02-41](http://noback.upyun.com/2020-12-02-14-02-41.png)


### 忽略文件
一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 `.gitignore` 的文件，列出要忽略的文件的模式
```bash
# cat .gitignore

# 表示要忽略所有以 .o 或 .a 结尾的文件
*.[oa] 

# 忽略所有名字以波浪符（~）结尾的文件
*~ 

# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```
gitignore的文档规范如下

- 所有空行或者以 # 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
- 匹配模式可以以（/）开头防止递归。
- 匹配模式可以以（/）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反

github上面有专门维护gitignore文件的项目 https://github.com/github/gitignore 当然特殊的需要自己加上去

##  比较diff

### 对比暂存与为暂存的内容
向之前有个`AM`状态的文件,这个文件中有一部分是暂存的有一部分是没有暂存的, 我们可以用`git diff `来对比比较文件之间的内容
![2020-12-02-14-11-10](http://noback.upyun.com/2020-12-02-14-11-10.png)

### 查看暂存起来的内容
![2020-12-02-14-15-32](http://noback.upyun.com/2020-12-02-14-15-32.png)

## 提交更新
提交更新文件,将暂存区的内容放到本地分支上去,为了区分每次提交的区别,需要写入标注,有两种方式
```bash
git commit # 这时候会打开一个编辑器来让你输入内容
git commit -m "update" # 直接输入标注
```
![2020-12-02-14-20-19](http://noback.upyun.com/2020-12-02-14-20-19.png)

## 移除文件
要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。 可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了

![2020-12-02-14-22-30](http://noback.upyun.com/2020-12-02-14-22-30.png)

保留本地文件,但是删除暂存区的文件
![2020-12-02-14-24-17](http://noback.upyun.com/2020-12-02-14-24-17.png)


## 查看提交历史
使用`git log`可以查看日志,这个命令很关键,回退或者多人合作开发的时候会用到
```bash
git log # 查看提交历史
git log --stats # 查看提交历史,展示修改的内容
git log --pretty=online # 查看提交实例,显示commit信息
# git log --pretty== # 还能添加很多的模式
git log --pretty=format:"%h - %an, %ar : %s" # 格式化输出
git log --author="xx"  # 仅显示作者匹配指定字符串的提交。
--since="2008-10-01" # 仅显示指定时间之后的提交。
--before="2008-10-01" # 仅显示指定时间之前的提交。
--committer # 仅显示提交者匹配指定字符串的提交。
--grep # 仅显示提交说明中包含指定字符串的提交。
```
![2020-12-02-16-06-22](http://noback.upyun.com/2020-12-02-16-06-22.png)
![2020-12-02-16-08-25](http://noback.upyun.com/2020-12-02-16-08-25.png)

格式化内容如下
```bash
%H 提交的完整哈希值
%h 提交的简写哈希值
%T 树的完整哈希值
%t 树的简写哈希值
%P 父提交的完整哈希值
%p 父提交的简写哈希值
%an 作者名字
%ae 作者的电子邮件地址
%ad 作者修订日期（可以用 --date=选项 来定制格式）
%ar 作者修订日期，按多久以前的方式显示
%cn 提交者的名字
%ce 提交者的电子邮件地址
%cd 提交日期
%cr 提交日期（距今多长时间）
%s 提交说明
```

## 撤销操作
使用第二次提交代替第一次提交

```bash
git commit -m 'initial commit'
git add forgotten_file
git commit --amend
```

取消暂存
```bash
git restore --staged <file>
```
![2020-12-02-16-16-53](http://noback.upyun.com/2020-12-02-16-16-53.png)

文件提交后修改内容,将修改后的文件回退到上一个版本
```bash
git restore <file>
```
![2020-12-02-16-19-38](http://noback.upyun.com/2020-12-02-16-19-38.png)


## 远程仓库的使用
查看远程仓库以及远程仓库地址
```bash
git remote
git remote -v
```
![2020-12-02-16-22-26](http://noback.upyun.com/2020-12-02-16-22-26.png)

添加远程仓库
```bash
git remote add pb https://github.com/paulboone/ticgit
```
![2020-12-02-16-23-48](http://noback.upyun.com/2020-12-02-16-23-48.png)
现在可以用pb来代替URL

### 从远程仓库拉取或者抓取
```bash
git fetch <remote>
```
![2020-12-02-16-28-32](http://noback.upyun.com/2020-12-02-16-28-32.png)
![2020-12-02-16-32-42](http://noback.upyun.com/2020-12-02-16-32-42.png)
这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。
如果你使用 clone 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 `origin` 为简写。 所以，`git fetch origin` 会抓取克隆（或上一次抓取）后新推送的所有工作。 必须注意 `git fetch` 命令只会将数据下载到你的本地仓库——它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。

### 推送到远程仓库
```bash
git push <remote> <branch>
```

### 查看某个远程仓库
```bash
git remote show origin
```


## git分支
![2020-12-02-16-36-53](http://noback.upyun.com/2020-12-02-16-36-53.png)
![2020-12-02-16-37-31](http://noback.upyun.com/2020-12-02-16-37-31.png)
![2020-12-02-16-37-56](http://noback.upyun.com/2020-12-02-16-37-56.png)
![2020-12-02-16-38-39](http://noback.upyun.com/2020-12-02-16-38-39.png)
![2020-12-02-16-40-01](http://noback.upyun.com/2020-12-02-16-40-01.png)
![2020-12-02-16-40-48](http://noback.upyun.com/2020-12-02-16-40-48.png)
![2020-12-02-16-41-20](http://noback.upyun.com/2020-12-02-16-41-20.png)


## Git 分支 - 分支的新建与合并
https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6

https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%AE%80%E4%BB%8B