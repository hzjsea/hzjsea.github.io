---
title: CI-CD实践
categories:
  - 技术补充
tags:
  - docker
  - CI 
  - CD
abbrlink: 4115d0cd
date: 2020-02-14 03:29:41
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [CI/CD 自动化部署流程](#cicd-自动化部署流程)
  - [gitlab](#gitlab)
  - [项目一 使用gitlab作为代码托管-实践shell](#项目一-使用gitlab作为代码托管-实践shell)
    - [搭建gitlab-server](#搭建gitlab-server)
    - [上传项目到gitlab地址](#上传项目到gitlab地址)
    - [gitlab-ci搭建](#gitlab-ci搭建)
  - [gitlab操作](#gitlab操作)
  - [项目二 使用gitlab作为代码托管-实践docker](#项目二-使用gitlab作为代码托管-实践docker)
  - [项目三 gitlab-ci + drf项目 搭建CI流程线](#项目三-gitlab-ci-drf项目-搭建ci流程线)
  - [错误](#错误)

<!-- /code_chunk_output -->
<!-- more -->


# CI/CD 自动化部署流程


## gitlab
gitlab-ci 另外的参数
https://docs.gitlab.com/ce/ci/yaml/README.html


## 项目一 使用gitlab作为代码托管-实践shell
gitlab-help文档
http://10.0.5.233:8888/help

### 搭建gitlab-server
安装git-lab

清华镜像源地址https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/
```bash
# cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 
# git --version
git version 1.8.3.1
# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-12.5.9-ce.0.el7.x86_64.rpm 
# ls
gitlab-ce-12.0.2-ce.0.el7.x86_64.rpm
```

安装依赖
```bash
sudo yum install -y git vim gcc glibc-static telnet
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd

sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```
安装gitlab
```bash
rpm -i gitlab-ce-12.0.2-ce.0.el7.x86_64.rpm 
```


### 上传项目到gitlab地址
```bash
➜  untitled git:(doc5) git push -u origin master
Username for 'http://gitlab.example.com:8888': root
Password for 'http://root@gitlab.example.com:8888':
Enumerating objects: 8414, done.
Counting objects: 100% (8414/8414), done.
Delta compression using up to 8 threads
Compressing objects: 100% (5158/5158), done.
Writing objects: 100% (8414/8414), 11.67 MiB | 732.00 KiB/s, done.
Total 8414 (delta 2103), reused 8401 (delta 2099)
remote: Resolving deltas: 100% (2103/2103), done.
To http://gitlab.example.com:8888/root/tc.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'. 
```

### gitlab-ci搭建
安装docker
```bash
curl -sSL https://get.docker.com/ | sh 
```
安装gitlab-ci-runner
```bash
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
sudo yum install gitlab-ci-multi-runner -y
```
查看运行状态
```bash
[root@manager55 ~]# sudo gitlab-ci-multi-runner status
gitlab-runner: Service is running! 
```
为了能让gitlab-runner能正确的执行docker命令，需要把gitlab-runner用户添加到docker group里, 然后重启docker和gitlab ci runner
```bash
[root@manager55 ~]# sudo usermod -aG docker gitlab-runner
[root@manager55 ~]# sudo systemctl restart docker
[root@manager55 ~]# sudo gitlab-ci-multi-runner restart 
```
注册CI-CD
```bash
[root@manager55 ~]# sudo gitlab-ci-multi-runner register
Running in system-mode.

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://gitlab.example.com:8888
Please enter the gitlab-ci token for this runner:
ppq_iFivNbgFgnusLE26
Please enter the gitlab-ci description for this runner:
[manager55]:
Please enter the gitlab-ci tags for this runner (comma separated):
tc
Whether to run untagged builds [true/false]:
[false]:
Whether to lock Runner to current project [true/false]:
[false]:
Registering runner... succeeded                     runner=ppq_iFiv
Please enter the executor: shell, ssh, virtualbox, docker-ssh+machine, docker-ssh, parallels, kubernetes, docker, docker+machine:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```
<font color='red'>如何获取token</font>

![2020-02-14-13-52-10](http://noback.upyun.com/2020-02-14-13-52-10.png)


编写gitlab.yml 任务流程
![2020-02-14-15-46-20](http://noback.upyun.com/2020-02-14-15-46-20.png)
```bash
# 定义 stages
stages:
    - build
    - test

# 定义 job
job1:
    stage: test
    tags:
        - tc
    script:
        - echo "I am jb1"
        - echo "I am in test stage"

# 定义 job
job2:
    stage: build
    tags:
        - tc
    script:
        - echo "I am jb2"
        - echo "I am in build stage"
```

提交gitlab.yml之后再去卡看CI/CD里面的pipeline，会发想任务正在进行
![2020-02-14-15-48-02](http://noback.upyun.com/2020-02-14-15-48-02.png)
通过所有的jobs,包括检验等等，并在CI服务器上执行script
![2020-02-14-15-48-58](http://noback.upyun.com/2020-02-14-15-48-58.png)
![2020-02-14-15-49-15](http://noback.upyun.com/2020-02-14-15-49-15.png)




## gitlab操作
修改端口,gitlab默认端口为8080端口，可以将它修改成其他的端口
```bash
sed  -r '/^external_url/s/com/com:8888/g' /etc/gitlab/gitlab.rb 
```
修改完配置文件之后要重启配置
```bash
sudo gitlab-ctl reconfigure 
```
查看一下启用状态
```bash
root:~# gitlab-ctl status
run: alertmanager: (pid 32102) 297s; run: log: (pid 30552) 532s
run: gitaly: (pid 32128) 296s; run: log: (pid 29584) 630s
run: gitlab-exporter: (pid 32151) 296s; run: log: (pid 30382) 550s
run: gitlab-workhorse: (pid 32166) 296s; run: log: (pid 30066) 574s
run: grafana: (pid 854) 240s; run: log: (pid 30748) 505s
run: logrotate: (pid 32218) 295s; run: log: (pid 30285) 562s
run: nginx: (pid 839) 241s; run: log: (pid 30184) 568s
run: node-exporter: (pid 32300) 294s; run: log: (pid 30327) 556s
run: postgres-exporter: (pid 32306) 294s; run: log: (pid 30633) 526s
run: postgresql: (pid 32316) 293s; run: log: (pid 29748) 624s
run: prometheus: (pid 32327) 293s; run: log: (pid 30489) 538s
run: redis: (pid 32347) 292s; run: log: (pid 29494) 636s
run: redis-exporter: (pid 32450) 292s; run: log: (pid 30432) 544s
run: sidekiq: (pid 743) 251s; run: log: (pid 30005) 580s
run: unicorn: (pid 1106) 214s; run: log: (pid 29965) 586s 
```

<!-- git remote remove https://gitee.com/Alpaca-H/tc.git -->



## 项目二 使用gitlab作为代码托管-实践docker
Password


## 项目三 gitlab-ci + drf项目 搭建CI流程线
1. 在搭建好的gitlab上，上传项目
2. 注册gitlab-ci-running
```bash
root:~/tc# gitlab-ci-multi-runner register
Running in system-mode.

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://gitlab.example.com:8888
Please enter the gitlab-ci token for this runner:
ppq_iFivNbgFgnusLE26
Please enter the gitlab-ci description for this runner:
[work233]:
Please enter the gitlab-ci tags for this runner (comma separated):
tcc
Whether to run untagged builds [true/false]:
[false]:
Whether to lock Runner to current project [true/false]:
[false]:
Registering runner... succeeded                     runner=ppq_iFiv
Please enter the executor: virtualbox, kubernetes, docker, docker-ssh, parallels, shell, ssh, docker+machine, docker-ssh+machine:
ssh
Please enter the SSH server address (e.g. my.server.com):
10.0.5.233
Please enter the SSH server port (e.g. 22):
22
Please enter the SSH user (e.g. root):
root
Please enter the SSH password (e.g. docker.io):
upyun123
Please enter path to SSH identity file (e.g. /home/user/.ssh/id_rsa):

Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```
编写gitlab-ci.yml
```bash
# 定义 stages
stages:
    - reploy
job1:
    stage: reploy
    tags:
        - tcc
    script:
        - echo "start reploy project about drf"
        # - sudo rm -rf /root/tc/
        # - git clone http://gitlab.example.com:8888/root/tc.git
        - cd /root/tc
        - cd /root/tc
        - sudo git fetch --all
        - sudo git reset --hard origin/master 
        - sudo git pull
        - sudo sed -i -r '/^ALLOWED/s#\[\]#\["\*"\]#' /root/tc/untitled/settings.py
        - source /root/tc/venv/bin/activate
        - echo "successed"
        # - cd /root/tc/
        - python3 manage.py runserver 0.0.0.0:8887
    
```

## 错误
1. 错误1，在创建CI的时候，没有在hosts中指定ip与域名的对应关系
```bash
ERROR: Registering runner... failed                 runner=ppq_iFiv status=couldn't execute POST against https://gitlab.com:8888/api/v4/runners: Post https://gitlab.com:8888/api/v4/runners: dial tcp 35.231.145.151:8888: i/o timeout
PANIC: Failed to register this runner. Perhaps you are having network problems 
```
解决:
```bash
echo 10.0.5.233  gitlab.example.com >> /etc/hosts
```

2. 权限的问题 gitlab-runner
当你创建完成gitlab-runner之后，系统会创建多个用户来提供服务
```bash
...
git:x:995:991::/var/opt/gitlab:/bin/sh
gitlab-redis:x:993:990::/var/opt/gitlab/redis:/bin/false
gitlab-psql:x:992:989::/var/opt/gitlab/postgresql:/bin/sh
gitlab-prometheus:x:991:988::/var/opt/gitlab/prometheus:/bin/sh
gitlab-runner:x:990:987:GitLab Runner:/home/gitlab-runner:/bin/bash 
```
运行gitlab-ci 时候，使用的是gitlab-runner用户的权限，有时候一些文件只有root才有权限执行，这里如果不切用户的话，可以给gitlab-runner用户附加root的权限
```bash
vi /etc/sudoers 

## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
gitlab-runner ALL=(ALL) ALL
```
<font color='red'>或者可以不使用shell的模式，在创建gitlab-running的时候使用ssh的模式</font>


3. git服务器上传内容之后，Gitlab的Pipelines一直在pending?
可能存在的几个问题，一个是网络的问题，项目包被pull到gitlab-runner上的时间过程
如果存在struct标签的话，应该是gitlab-runner被暂停了
解决办法:
重启gitlab-runner就可以了