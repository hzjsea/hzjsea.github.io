---
title: git基础2
categories:
  - 技术补充
tags:
  - git
abbrlink: 9891ebc8
date: 2019-12-29 20:50:50
---

<!--more-->

# git介绍
Git是一个分布式版本管理系统，是为了更好地管理Linux内核开发而创立的。
Git可以在任何时间点，把文档的状态作为更新记录保存起来。因此可以把编辑过的文档复原到以前的状态，也可以显示编辑前后的内容差异。

笔记来自 https://backlog.com/git-tutorial/cn/intro/intro1_2.html
        https://git-scm.com/docs

> 优化提醒
> 1. 不同类别的修改 (如：Bug修复和功能添加) 要尽量分开提交，以方便以后从历史记录里查找特定的修改内容。
> 

## 数据库
git数据库一共分为本地数据库和远程数据库
- 远程数据库: 配有专用的服务器，为了多人共享而建立的数据库
- 本地数据库: 为了方便用户个人使用，在自己的机器上配置的数据库。


![2020-12-02-11-34-40](http://noback.upyun.com/2020-12-02-11-34-40.png)


## 流程
工作区(我们当前的文件夹)
暂存区(add 提交之后文件所存在的位置)
仓库(commit 之后将暂存区提交到的地方)

首先是init初始化我们的工作区，这时候当前存在.git文件的就是我们的工作区，当前路径为我们的根目录
在工作区创建一个新的文件test.txt， 
git add 之后文件上传到暂存区，我们可以用git status 会记录当前暂存区的操作
git diff可以比较工作区和暂存区的差异，当然这里是没有任何差异的，因为他们是同样的两份文件
修改test.txt 之后再去git diff 你会发现他们之间存在着差异，因为这个时候他们已经不是同样两份文件了
假如你再次 使用Git add . 再用git diff 时又没有差异了


## 修改记录的提交
若要把文件或目录的添加和变更保存到数据库，就需要进行提交
每次提交之后,都会生成一段log ,以时间顺序排列状态被保存到数据库中,这段log会根据修改的内容计算出没有重复的40位英文及数字(id)来给提交命名


## git的配置
Git 自带一个git config的工具来帮助设置控制 Git 外观和行为的配置变量。
这些变量存储在三个不同的位置
- `/etc/gitconfig` 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果在执行 git config 时带上 `--system` 选项，那么它就会读写该文件中的配置变量
- `~/.gitconfig` 或 `~/.config/git/config` 文件：只针对当前用户。 你可以传递 `--global` 选项让 Git 读写此文件
- `.git/config` 这个是针对当前仓库的,你可以传递`--local`选项让 Git 强制读写此文件，虽然默认情况下用的就是它

越底层,覆盖的等级越高,所以他们覆盖的顺序是
```
.git/config > ~/.gitconfig > /etc/gitconfig
```

查看对应的git配置
```bash
git config --list --show-origin
file:/usr/local/etc/gitconfig   credential.helper=osxkeychain
file:/Users/alpaca/.gitconfig   filter.lfs.clean=git-lfs clean -- %f
file:/Users/alpaca/.gitconfig   filter.lfs.smudge=git-lfs smudge -- %f
file:/Users/alpaca/.gitconfig   filter.lfs.process=git-lfs filter-process
file:/Users/alpaca/.gitconfig   filter.lfs.required=true
file:/Users/alpaca/.gitconfig   user.email=user@gmail.com
file:/Users/alpaca/.gitconfig   user.name=user
file:/Users/alpaca/.gitconfig   credential.helper=store
```


## git基础
### 安装
```bash 
# ubuntu
sudo apt-get install git
# centos 
yum install git -y
# mac 
brew install git
```

## 全局声明用户
安装完git之后就需要设置用户信息
```bash
git config --global user.name "your name"
git config --global user.email "xxx@xxx.com"
```
设置全局`--global`之后,git会全局设定用户信息,如果需要制定特定的用户信息,可以在仓库中设置
```bash
git config  user.name "your name"
git config  user.email "xxx@xxx.com"
git config --list --show-origin
# file:.git/config        user.name=xx
# file:.git/config        user.email=zz
# 查看当前仓库的用户信息
git config user.name
# xx
```
这样当前仓库的信息就会被更改,因为当前仓库的gitconfig覆盖等级是最高的



## 声明仓库
声明仓库有两种方式,一种是clone现成的仓库,一种是自己定义一个仓库,
两种方式最后都会在本地创建一个仓库
```bash
# clone
git clone online/xx.git
# init 
mkdir -p project && cd project && git init
```

## 文件追踪
初始化好以后的仓库,文件是没有被追踪的
可以通过`git add`命令来指定所需的文件来进行追踪，然后执行`git commit`



## 提交并提供信息
将文件提交到仓库
```bash 
git commit -m "your message"
```

## 添加文件
```bash
git add xxx.txt
git add . #添加当前文件夹所有文件 
``` 

## 查看状态
git status

```bash
# 添加文件后的状态
 alpaca@hzj  ~/hzj/tu   master ●  git add README.md
 alpaca@hzj  ~/hzj/tu   master ✚  git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   README.md


# 提交 commit 后的状态
 alpaca@hzj  ~/hzj/tu   master ✚  git commit -m "update some file"
[master ff070f9] update some file
 1 file changed, 2 insertions(+)
 alpaca@hzj  ~/hzj/tu   master 

# 状态
  alpaca@hzj  ~/hzj/tu   master  git status
On branch master
nothing to commit, working tree clean
```

## 查看日志
当你每一次使用commit 将文件提交到当前分支的时候，记录下每一个步骤
查看使用
```bash
# 查看日志
git log

commit bb3c0e3a1e850af0c70d3ce3a22995eaae177e82 (HEAD -> master)
Author: alpaca <1097690268@qq.com>
Date:   Thu Nov 21 11:46:24 2019 +0800

    update

commit ff070f92dee18935752e5a98b2310d275e1191b6
Author: alpaca <1097690268@qq.com>
Date:   Thu Nov 21 11:42:58 2019 +0800

    update some file

commit 230660d585a95b90e3c6176ad21664c1b10f746a
Author: alpaca <1097690268@qq.com>
Date:   Thu Nov 21 10:37:04 2019 +0800

    update some markdown file


# 仅查看版本号
git log --pretty=oneline
# 指定版本号 如果你装了zsh 按tab就会出来commit_id
git log commit_id 
```
由随机码(版本号) + 作者 + 日期 + commit_message 组成


## 回退操作
版本回退，首先我们要知道一个事情 git log 只会记录下commit的过程，也就是上传到当前分支的过程，因此回退操作他并不会记录，但是他会重载之前的log状态，也就是当你一共提交了3次内容，你回退到了第三次提交前的版本。那么你的第三次提交log并不会被记录
```bash 
# 回退add操作  checkout -- file  暂存区 --> 工作区
git checkout -- file

# 回退connit操作上  仓库 ---> 暂存区
git reset --hard commit_id 

# 查看回退日志
git reflog
```


## 提交远程仓库
之前的操作都是本地的，包括最后一步git commit 也只是提交到了本地的仓库。为了更好的保存内容，我们已经引入云仓库或者说托管我们的项目
```bash
# 添加远程仓库地址
git remote add origin https://gitee.com/Alpaca-H/how-to-learn-git.git 

# 提交当前分支到远程仓库
git push  origin master

# 建议第一次使用push -u
# Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来 在以后的推送或者拉取时就可以简化命令。
```

## 从远程仓库下载
```bash 
git clone https://gitee.com/Alpaca-H/how-to-learn-git.git
```


第一次提交的存在的问题
```bash
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://gitee.com/Alpaca-H/how-to-learn-git.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details. 
```
如果你是在 码云上创建的项目，它本身就存在两个README文件，也就是说仓库中本身就存在文件了，这时候你删掉这两个也没用，(github上好像没这个问题)
我们需要同步远程仓库，也就是拉取下来并用--allow-unrelated-histories 合并

```bash
git pull origin master --allow-unrelated-histories   
git commit -m "xxx" #重新提交
git push origin master
```

## 从远程仓库下载 -- 指定分支 
```bash
git clone -b hzj_branch https://gitee.com/Alpaca-H/how-to-learn-git.git 
```


## 分支管理

```bash
# 创建分支  
git branch hzj
# 切换分支
git checkout hzj

# 创建并切换分支
git checkout -b hzj

# 查看当前分支
git branch
```

记住上面的只是分支管理，他只是在你的本地有了分支，在你没有push之前所有的一切都是本地操作

## 分支管理2
```bash
# 切换到新的分支 
git swicth -c dev
# 切换到已有分支
git swicth dev
```


```bash
# 添加文件
echo "我是hzj分支" > README2.md
git add README2.md
git commit -m "update hzj"
git push -u origin  hzj # 提交到hzj的分支
```

## 合并分支
进入master
```bash
git  merge hzj # 合并在本地仓库的文件
```

## 删除分支
```bash
git branch -d hzj 
```


## 强制覆盖本地代码
```bash
$ git fetch --all
$ git reset --hard origin/master 
$ git pull 
```

## git 关于Git每次进入都需要输入用户名和密码的问题解决
```bash
cd your-project
git config --global credential.helper store 
git pull
```
输过一遍以后就不用再输入了


## 分支同步主干
```bash
# 切换到master分支
git checkout master 
# 下拉master
git pull origin master
# 切换分支
git checkout hzj
# 同步分支
git merge  master
```

## 主干同步分支(慎用)
```bash
# 切换到主分支
git checkout master
# 同步分支
git merge hzj
# 提交
git push origin master 
```

## inventory操作
1. 拉取项目
git clone https://gitlab.s.upyun.com/infrastructure/inventory.git
2. 查看当前分支
git branch
3. 切换分支
git checkout hzj
4. 修改内容
...
5. 上传
git commit -am "update"
git push 
到web端创建合并请求
6. 更新
首先切换到master，
拉取最新git pull
切换次分支 git checkout hzj
合并更新分支到hzj分支 git merge origin hzj

发消息给有master权限的人，交由master来合并


## 错误整理

### 场景1 
之前用过一个Alpaca-H的github账号，后来被弃用了，然后换成hzjsea的账号，并且使用git config --global更新了username和email也不行
后来发现mac会在本地生成一份凭证，存储在本地

报错
```bash
➜  sea_blog git:(master) git push  origin master
remote: Permission to hzjsea/sea_blog.git denied to Alpaca-H.
fatal: unable to access 'https://github.com/hzjsea/sea_blog.git/': The requested URL returned error: 403
```

解决办法
1. 进入Keychain Access (不知道在哪儿的可以command+space查找)
2. 在搜索框输入'git'进行查找，将找到的文件删掉，这里保存了历史账号的信息
3. 删除之后重新用git config --global更新username和email即可，之后git push会要求你输入username和password
4. done!

![2020-03-14-23-38-56](http://noback.upyun.com/2020-03-14-23-38-56.png)

### 场景2
经常要输账号密码

![2020-04-10-14-47-32](http://noback.upyun.com/2020-04-10-14-47-32.png)
解决办法
```bash
git config --global credential.helper store
```
生成一个本地文件用于存放账号密码，然后再pull一下就行了