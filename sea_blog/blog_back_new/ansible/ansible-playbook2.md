---
title: ansible-playbook合集
categories:
  - ansible
tags:
  - ansible
abbrlink: ba9090fe
date: 2020-05-11 18:05:15
---


<!-- more -->
# ansible-playbook合集

## shell

### ansible利用shell结果注册变量

<font color='red'>这里不只是使用于shell模块，也适用与command模块</font>

```bash
---
  - name: start servers
    shell: "{{ prefix_path }}/{{ role }}/server.sh start" 
    register: info
  - name: check Asset
    shell: "{{ prefix_path }}/{{ role }}/checkAsset.sh"
    when: info.stdout.find("OK") != -1
  - name: check Safe
    shell: "{{ prefix_path }}/{{ role }}/checkSafe.sh"
    when: info.stdout.find("OK") != -1
  - name: test result
    command: "echo {{ info }}"
```

这里面是先执行`./server.sh start` 来获取返回结果 注册`register`到info这个变量
取值是`info.stdout`
在ansible里面 when后面是可以跟python语句的，后面就是找有没有ok这个str,有的话就执行，没有的话就不执行

