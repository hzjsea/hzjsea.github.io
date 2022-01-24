---
title: git
date: 2021-11-29 17:09:05
categories:  [git]
tags: [git]
---


<!--more-->


# git

手册:
https://www.git-tower.com/learn/git/ebook/cn/command-line/introduction#start
https://git-scm.com/book/zh/v2
https://www.cnblogs.com/lzkwin/p/12658029.html


git操作查询手册

> git命令的帮助列表

```bash
➜  ~ git

start a working area (see also: git help tutorial)
   clone     Clone a repository into a new directory
   init      Create an empty Git repository or reinitialize an existing one

work on the current change (see also: git help everyday)
   add       Add file contents to the index
   mv        Move or rename a file, a directory, or a symlink
   restore   Restore working tree files
   rm        Remove files from the working tree and from the index

examine the history and state (see also: git help revisions)
   bisect    Use binary search to find the commit that introduced a bug
   diff      Show changes between commits, commit and working tree, etc
   grep      Print lines matching a pattern
   log       Show commit logs
   show      Show various types of objects
   status    Show the working tree status

grow, mark and tweak your common history
   branch    List, create, or delete branches
   commit    Record changes to the repository
   merge     Join two or more development histories together
   rebase    Reapply commits on top of another base tip
   reset     Reset current HEAD to the specified state
   switch    Switch branches
   tag       Create, list, delete or verify a tag object signed with GPG

collaborate (see also: git help workflows)
   fetch     Download objects and refs from another repository
   pull      Fetch from and integrate with another repository or a local branch
   push      Update remote refs along with associated objects
```

## 基本操作

默认分支是master

```bash
# 创建分支
git branch -a # 列出所有的分支， 包括远程分支和本地分支
git branch local_name # 创建一个本地分支
git checkout -d local_name # 删除本地分支  删除时最好不要在当前分支
git checkout -D local_name # 强制删除本地分支
git checkout -dr origin/<branch_name> # 删除远端的分支
git push --set-upstream origin origin_name # 创建本地分支与远端分支连接
git init . # 初始化环境
git clone https://xxx
git clone https://xxx.git target_name # 克隆并且重命名
git checkout local_name # 签出到本地的local_name分支

git checkout -b local_name # 在本地创建环境，和远端无关, 会复制一个当前分支 比如你在master中使用， 会复制master分支并创建一个local_name的本地分支
git checkout -b local_name origin/origin_name # 复制一份远端的分支，并在本地重命名成local_name

git commit --amend -m "This is the correct message" # 修改暂存区当中的最后一次提交批注

git checkout -- file/to/restore.ext # 撤销本地改动,当改动还没有被提交之前,它们仍然被称之为"本地"改动

git reset --hard HEAD # 恢复到上次提交之后的版本, 恢复是正向的，该操作无法逆向

```

### 撤销已提交的改动

https://blog.csdn.net/yxlshk/article/details/79944535

- git revert

git revert是用于“反做”某一个版本，以达到撤销该版本的修改的目的。比如，我们commit了三个版本（版本一、版本二、 版本三），突然发现版本二不行（如：有bug），想要撤销版本二，但又不想影响撤销版本三的提交，就可以用 git revert 命令来反做版本二，生成新的版本四，这个版本四里会保留版本三的东西，但撤销了版本二的东西

![](https://noback.upyun.com/2021-12-01-11-33-56.png!)

```bash
# 测试
for i in {1..5};do touch master${i} ; done
for i in {1..5};do git add master${i}; git commit -m "master${i}" ; done
git revert 4dbe2e01cbcd6e8d6b531811bb56c
```




- git reset

git reset的作用是修改HEAD的位置，即将HEAD指向的位置改变为之前存在的某个版本

```bash
# 退到上一次改动的时候
git reset --hard
# 退到某一个目标版本号
git reset --hard 目标版本号
```

![](https://noback.upyun.com/2021-12-01-11-36-17.png!)

## 文件状态

一般情况下在 Git 中文件有两种状态：

> 未被追踪的文件(untracked): 如果这个文件还未被纳入版本控制系统中，我们称之为“未被追踪的文件”。这就表示版本控制系统不能监视或者追踪它的改动。一般情况下未被追踪的文件会是那些新建的文件，或者是那些没有被纳入版本控制系统中的忽略文件。

> 已追踪的文件(tracked):所有那些已经被纳入版本控制系统的项目文件我们称之为“已被追踪的文件”。Git 会监视它的任何改动，并且你可以提交或放弃对它修改。

查看已经跟踪的文件
```bash
# git ls-tree  -r <branch> --name-status
git ls-tree  -r master --name-status
```

查看未被跟踪的文件
```bash
git status

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        new
```

将未跟踪的文件添加到已跟踪的文件中去
```bash
git add . # 添加当前文件夹中的所有 
git add <file_name> # 添加指定file
```

不建议使用`git add .` 的方式使用，一次提交只对映一个相关的改动

未跟踪文件白名单，通过`.gitignore`文件会把一些文件名列为白名单，这些文件在add的过程不会被添加到已跟踪的文件到中去
```bash
The following paths are ignored by one of your .gitignore files:
hello
Use -f if you really want to add them.
```

### 暂存区
第一次模拟提交 会把已跟踪的文件添加到暂存区当中去

<div style='font-size:20px;color:red'>暂存区是一个很重要的地方，他会暂存当前分支中的内容，如果你没有把当前修改的内容提交到暂存区的话，你切换分支后本地的改动也会被带着去到其他分支，这样久而久之就会出现很严重的问题，所以保持你的提交只在独立分支环境下是很有意义的</div>
</br>



> 多次提交，多次批注信息
因为每次提交到暂存区之后，可以使用`commit -m ""`提交一些批注
因此对每一份改动都必须对应一份批注

黄金法则1

```bash
No. 1: 一次提交只对映一个相关的改动
当你想要进行提交时，非常重要的一点就是，它只能包含一个相关的改动。也就是说，不要在一次提交中包含一些和提交目的无关的改动，每一种不同的改动要打包在不同的提交中。比如，在一次提交中既添加了一个新的功能（例如：用户注册功能），又包括了对一些错误的修复（例如：修复了Bug#122):
但如果把它们两个功能和在一起作为一个批注(例如: add new func and fix bug#122) 就显得不怎么合理

理解这些复杂的提交对于开发团队里的其他成员来说是非常困难的。可能过了一段时间连你自己也糊涂了。你的队友不得不先要搞明白这个错误，弄清楚你为什么要同时修复它，之后才能试图了解这个新添加的用户注册功能。
不可能复原或者撤销其中一个改动。试想一下，那个新添加的用户注册功能可能存在很严重的错误，你想撤销它但是你又不想撤销那个修复错误的改动。你该怎么办呢？
也就是说，一次提交一定要包括一个且仅仅只能包括一个和提交目的相对映的改动：修复两个错误（最起码的）你要进行两次不同的提交。或者当开发一个新的非常庞大的功能时，你必须要把它分成几个小的并且在逻辑上有意义的提交，这样做可以有效地减少产生错误的可能性。

这种小的提交可以让开发团队的其他成员更好地理解这个改动。一旦发现这个改动有问题，就可以非常简单地撤销，而不会影响到其它的功能。
```

![](https://noback.upyun.com/2021-11-30-10-55-51.png!)

![](https://noback.upyun.com/2021-11-30-11-13-31.png!)

> 暂存区操作

```bash
# 将文件添加都暂存区当中
git add example2
# 将文件从暂存区当中移除 并删除该文件
git rm example2 -f
# 将文件从暂存区当中移除 并保留该文件
git rm example2 --cached
```

### 关于批注
提交的时候需要写好批注

```bash
No. 2: 高质量的提交注释
花一点时间写一个好的提交注释是非常值得的，这样可以让开发团队的其他成员非常容易地明白你做这次提交的目的和你的改动（过了一段时间对你自己也有帮助）。


针对你的改动写一个简短的注释标题（原则上不要超过50个字符），然后使用一个空行来分隔注释的内容。注释的内容要尽可能的详细并且要能回答以下几个问题：为什么要做这次修改？与上一个版本相比你到底改动了什么？
```

```bash
# 提交短批注
git commit -m "{}"
# 提交长批注
git commit 
```

```bash
什么才是一个好的提交
一个高质量的手动提交对你的项目和你自己是非常有意义的。什么才是一个好的提交呢？在这里有一些基本的原则：

提交要仅仅对应一个相关的改动
首先，一次提交应该仅仅只对应一个相关的改动。不要把那些互相毫无关联的改动打包在同一次提交中。如果这次提交出现了什么问题，解决和撤销它将是非常困难的。
完整的提交
千万不要提交那些没有完全完成的改动。如果你想要临时保存一下你当前的工作，例如一个类似于剪贴板 （clipboard） 的功能，你可以使用 Git 提供的 “Stash” 功能（这个功能我们将在本书之后的章节为大家介绍）。但是一定不要直接提交它。
提交前测试
当你提交你的改动时，不要理所当然地认为你的改动永远正确。在你提交你的改动到你的仓库前，进行有效的测试是非常重要的。
高质量的提交注释
一次高质量的提交需要一个好注释。请参考本章之前的章节 “高质量的提交注释”。
最后，你须要养成一个频繁地进行提交的习惯。这样做将自然而然的让你避免一个很庞大的提交，并且使这些提交可以更好只对映一个相关的改动
```

## 提交历史
git每次提交都会记录一次数据，这里的提交指的是push成功 ，而并非是commit 提交到暂存区的这一步, 这里粘贴部分
```bash
commit 034daaa83ac4a30f96f8e02bb1ffc606c6263b0e (HEAD -> ccy#3)
Author: hzjsea <hzjsea@gmail.com>
Date:   Tue Nov 30 11:16:45 2021 +0800

    echo some commit

commit 7381c3b1065409aa1b1482d627443d73519ef386 (origin/ccy#3)
Author: hzjsea <hzjsea@gmail.com>
Date:   Tue Nov 30 11:03:45 2021 +0800

    add example file

commit 65df57e37e4db95ebfe85883d3f19db36622a068
Author: hzjsea <hzjsea@gmail.com>
Date:   Tue Nov 30 10:55:32 2021 +0800

    add 吃饭 action

commit c1242cbe7ffea5bc6c5b376ba1aba3dda8a509d4
Author: hzjsea <hzjsea@gmail.com>
Date:   Tue Nov 30 10:55:03 2021 +0800

    add 喝水 action

commit 720272bbcde23865df679960626704587b683032
Author: hzjsea <hzjsea@gmail.com>
Date:   Tue Nov 30 10:54:20 2021 +0800

    add new

commit 606c13b89bec46c36e56cda30f607919fed11aab
Author: hzjsea <hzjsea@gmail.com>
Date:   Tue Nov 30 10:53:59 2021 +0800
```
有如下几个字段
- commit hash 标识唯一
- author name & email
- date
- commit Message 提交批注


## 分支
上面的内容你都可以在单分支中完成， 但是在实际开发中引入了分支的概念，也就是你和你的小伙伴在不同的分支上并行工作，最后合并。这样就可以大大的提高工作效率
![](https://noback.upyun.com/2021-11-30-11-32-42.png!)

```bash
# 创建本地分支
git branch new_master
# 查看详细的分支详情 最后一次提交ID
#   hzj    772e7f3 touch master
#   hzj#1  772e7f3 touch master
#   hzj#2  772e7f3 touch master
#   master 0812411 update
git branch -v 
```
创建本地分支后会在当前分支复制一份出来
```bash
<master> git branch new_master
git branch -v 
```
![](https://noback.upyun.com/2021-11-30-11-43-16.png!)
会发现master和new_master的分支详情中 最后一次提交的hash id 是一样的

> 在当前分支如果有未提交的改动的话，是无法切换分支的

```bash
No. 4: 不要提交一个还未完成的工作
不要提交一个还未完成的工作，但这也并不意味着在提交前你必须要完成这个功能定义的所有需求。恰恰相反，对于一个很大的功能模块来说，要把它正确分割成小的完整的逻辑块，用来进行频繁的提交。但是，千万不要为了得到一个干净的工作副本（working copy）而提交一些不完整的改动。在这种情况下，你可以考虑使用 Git 提供的 “Stash” 功能。
```
解决的方式是
1. 完成未完成的工作，提交一次请求后切换分支
2. 舍弃当前分支的修改 `git checkout .`
该方法指的是将某一个文件回到最近一次`git commit`或`git add`时的状态, 单文件是 `git checkout -- readme.txt`,`.`表示所有文件
3. 使用stash功能


## 暂时更改保存


## 签出

git中的分支从初始化开始以master为起点，生成一条记录提交记录的链，复杂的链如下所示
图中是一部分gin-admin库的链, 分支越多，可以预示着项目也越大，也预示着参与该项目的人员越多
![](https://noback.upyun.com/2021-11-30-17-40-13.png!)

签出的本质是在当前分支复制一份做一个新的起点，该分支会另起一个主题，后期可以选择是否并入主链

### 合并改动
![](https://noback.upyun.com/2021-12-01-10-32-10.png!)
![](https://noback.upyun.com/2021-12-01-10-31-41.png!)
合并的过程中会另外开启一个节点做合并操作

### 处理合并冲突

### 分支分类-短期分支与主题分支
根据分支的使用场景和使用流程，可以大致分为两种类型的分支
- 短期分支(主题分区) (例如修复某一分支的bug, 维护某一主题功能)
- 长期分支

短期分支(主题分区): 单一主题的长期分支,而且它包含的代码要和它的主题相对应。例如，你不应该建立一个关于购物车功能的分支，并且再在这个分支上去提交一些有关于邮件订阅功能和错误修复的改动，它们都只有非常短暂的生命周期。通常情况下，这个生命周期只维持到这个开发主题的结束之后（例如当错误被修复，新功能被完成……），这个分支的改动就会被整合到项目的大环境中去，并且这个分支也会被随之删除掉

每当你开始着手开发一个新的功能的或是修复错误时，你都应该对应不同的主题建立一个新的分支。这是一种很通用的做法，并且也要成为你的一种习惯。

如果在你的项目仓库中只存在一个长期分支，那么所有的主题分支都必须基于这个 “master” 分支。当你所开发的功能完成后，或者是错误被修复后，这些所对应的改动就理所当然的要被合并回 “master” 分支。

在你开发你的新功能的同时，团队的其他开发人员已经把各自完成的改动整合回了 “master” 分支。在这种情况下，你就必须经常性地把那些在 “master” 分支上的改动合并到你的工作分支上来。这就确保了你的工作分支一直处于最新的状态,并且当你要把已完成的改动整合回 “master” 分支上时，可以减少可能出现的冲突和风险。

请不要忘记这个简单的黄金法则：只有正确和稳定的代码才能被整合到 “master” 分支上来！如何确保这些代码正确性呢？这就依靠你和你的团队了。例如使用单元测试（unit tests），代码审查（code reviews）等等。

![](https://noback.upyun.com/2021-12-01-10-39-39.png!)

长期分支: 相对于那些基于一个单一开发主题或是一个错误修复的短期分支来说，这类分支被定义在更高的层面上。它们代表了在整个项目生命周期的一种状态（比如：“产品”，“测试”，“研发”状态）。它们在项目中保存的时间比较长，甚至可能是整个项目的生命周期。这类分支有如下一些典型特点：

你不应该直接在这个分支上工作。但是你可以整合其他的分支（例如功能分支或是其他的长期分支）到这类分支中来，尽量不要对它直接进行提交。
一般来说，在长期分支之间也存在不同的等级。例如 “master” 分支一般被定义为最高等级。它应该只保存产品代码（production code）。在它之下应该存在一个 “开发（development）” 分支。它会被用来进行真正的开发和测试工作，然后整合入 “master” 分支……

> 在一般的情况下， “master” 分支可以有效地代表你的产品代码。然而一个重要的前提就是，所有被合并到 “master” 分支的代码都必须保证正确！ 你必须保证它们的质量。因此它们必须经过测试，它们的代码必须被检验过和确认过。

### 解决合并冲突2

冲突常见问题标记如下
```bash
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g. 'git pull ...') before pushing again
```
意思就是说你要在提交你的分支之前先要拉取远程的分支中新的内容
这里有几种对应的解决方法

1. git checkout . 丢弃本地分支上的修改到最近一次commit的状态
可能会出现无效的输出
```bash
UPdated 0 paths from the index 
```
2. 如果希望保留生产服务器上所做的改动，仅仅并入新的配置项
```bash
git stash
git pull
git stash pop
```

3. 如果希望用代码库中的文件完全覆盖本地工作版本
```bash
# 1. 回到远程服务器中最新的一个版本
git reset --hard
git pull
```


## 远程仓库
![](https://noback.upyun.com/2021-12-01-10-52-40.png!)

跟踪仓库
https://www.git-tower.com/learn/git/ebook/cn/command-line/remote-repositories/inspecting-remote-data#start


## 扩展
上面的内容还是基础的内容，但是在多分支作业下，还是需要做到很多方面的内容

大致上的分支为
- 员工分支1 
- 员工分支2
- 员工分支3
- dev分支
- master主分支

在保证项目目录完整的情况下， 员工分支负责各自的工作模块，完成之后将内容合并到dev分支验收
dev分支验收之后再交由项目负责人合并到master分支
原则上不管员工分支的内容如何，都必须保证dev分支的干净

## 问题

> fatal: 'origin/x' is not a commit and a branch 'x' cannot be created from it

原因如下
```bash
git branch -b x origin/x
```
远端分支必须要指定已经存在的分支

> 合并冲突解决

出现的原因是因为 假设员工 Member A  和员工 Member B 
同时修改改了文件A B C 但最终在代码检查的时候 以A员工的分支为标准，
所以员工B要去合并员工A的内容，这时候就会出现下面的问题
```bash
(venv) ➜  jing-platform git:(hzjsea_cur) git merge image_acquisition    
Auto-merging startup-sample.py
CONFLICT (content): Merge conflict in startup-sample.py
Auto-merging sg_auth/login_views.py
Auto-merging app_plugins/image_acquisition/views.py
CONFLICT (content): Merge conflict in app_plugins/image_acquisition/views.py
Auto-merging app_plugins/hik/variables.py
CONFLICT (content): Merge conflict in app_plugins/hik/variables.py
Automatic merge failed; fix conflicts and then commit the result.

```
主要的标志就是 Automatic merge failed 
有两种解决办法，一种是用界面去操作，这样在合并的时候会直接报错让你解决
<p style="color:red">最好用这种方式 </p>

第二种解决办法就是在command line的方法去解决，会在对应的文件中出现如下的内容
```bash
<<<<<<< HEAD
    current_exit_code = MainWindow.EXIT_CODE_REBOOT
    # https://stackoverflow.com/questions/34372164/how-do-i-restart-my-pyqt-application
    while current_exit_code == MainWindow.EXIT_CODE_REBOOT:
        freeze_support()
        app = QApplication(sys.argv)
        serverName = release.identity
        socket = QLocalSocket()
        socket.connectToServer(serverName)
        if socket.waitForConnected(500):
            notify_and_quit(app)
        localServer = QLocalServer()
        localServer.listen(serverName)
        s = Splash("%s%s" % (release.app_name, release.year))
        s.showMessage("System starting, please wait...", 0x44, QColor(255, 255, 255))

        app.processEvents()


        init_database_management()
        dlg = LoginView("登录")
        dlg.exec_()

=======
    # from PyQt5.QtNetwork import QLocalSocket, QLocalServer
    # from sg_common import Splash
    # import time

    freeze_support()
    app = QApplication(sys.argv)
    serverName = release.identity
    socket = QLocalSocket()
    socket.connectToServer(serverName)
    if socket.waitForConnected(500):
        notify_and_quit(app)
    localServer = QLocalServer()
    localServer.listen(serverName)
    s = Splash("%s%s" % (release.app_name, release.year))
    s.showMessage("System starting, please wait...", 0x44, QColor(255, 255, 255))
    app.processEvents()
    init_database_management("")
    dlg = LoginView("")
    if dlg.exec_() == QDialog.Accepted:
>>>>>>> image_acquisition
        main = MainWindow(app, s, local_server=localServer)
        build_str = hasattr(release, "build_time") and release.build_time or str(datetime.datetime.today())[:19]
        main.setWindowTitle("%s - %sV%s - [build %s]" % (release.app_name, release.year, release.version, build_str))
        # try:
        #     import psutil
        #
        #     local_machine_handle = winreg.OpenKey(winreg.HKEY_LOCAL_MACHINE, "SOFTWARE")
        #     app_info = winreg.CreateKey(local_machine_handle, "Sinorld_Technologies")
        #     winreg.SetValueEx(app_info, "app_name", 0, winreg.REG_SZ, release.name)
        #     winreg.SetValueEx(app_info, "app_identity", 0, winreg.REG_SZ, release.identity)
        #     pid = os.getpid()
        #     p = psutil.Process(pid)
        #     winreg.SetValueEx(app_info, "app_path", 0, winreg.REG_SZ, os.path.join(os.getcwd(), p.name()))
        #     winreg.SetValueEx(app_info, "app_pid", 0, winreg.REG_SZ, "%s" % pid)
        #     winreg.SetValueEx(app_info, "app_pipe", 0, winreg.REG_SZ, str(localServer.fullServerName()))
        #     local_machine_handle.Close()
        # except Exception as e:
        #     _logger.error("Operation RegisterKey Error:%s", e)
        screen = QDesktopWidget().screenGeometry()
        if config["display_mode"] == "fullscreen" and screen.width() == 1024 and screen.height() == 768:
            main.showFullScreen()
        else:
            main.show()
        s.finish(main)
        main.exec_defered_funcs()
        sys.exit(app.exec_())

```

简化的来看就是这样的
```bash
==========HEAD
此为B员工的
==========
此为A员工的
>>>>>>>>> A 
```
如果要保留A员工的内容 删除B的就行
```bash
# ==========
此为A员工的
# >>>>>>>>>>>
```

第三种解决办法比较暴力，删除B员工修改的冲突的文件，然后合并
丢弃的方式有
```bash
rm -f
git chckout .
```


## 法则

保持远程同步
在 Git 中，远程和本地仓库可能彼此毫无依赖关系。不管怎样，本地和远程的分支必须被一致对待。

但是这样也并不代表你必须要发布你的 每一个 本地分支，拥有一些完全属于你的私有分支也是非常必要和有意义的。例如当你单独的研究一些新功能，或是尝试调试一些新的技术时。
不管怎样，当你要发布你的一个本地分支时，你就应该给它对应的远程分支一个相同的名称。例如，如果你有一个本地分支被命名为 “login-ui”，当你要发布它时，在远程仓库上它也必须拥有 “login-ui” 这个名字。

频繁推送
保持与远程分支的同步并不是只停留在结构层面上，经常性地通过 “git push” 命令发布你的改动可以有助于团队里的其他的开发人员看到和使用你的最新开发成果。附带的还有一个最大的好处是，它可以作为你的远程备份。

## 优秀的分支策略 git-flow
https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/git-flow



