---
title: ^python装饰器以及使用场景
date: 2021-01-29 16:55:43
categories:  [python]
tags: [python]
---


<!--more-->


# ^python装饰器以及使用场景


> 装饰器无参数 


```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: gg.py
@time: 2021/1/29 3:47 下午
'''

def good_future(func):  # 执行顺序1
    def wrapper():      # 2
        print("1")      # 4
        func()          # 5
        print("2")      # 7
    return wrapper      # 3


@good_future
def buy():
    print("2.5")        # 6


if __name__ == '__main__':
    buy()

# 1 
# 2.5
# 2
```

> 装饰器有参数

```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: gg.py
@time: 2021/1/29 3:47 下午
'''


def decorator(func):
    def inside(lt: list):
        try:
            func(lt)
            print('排序结束')
        except TypeError as e:
            # print(e)
            pass
    return inside


@decorator
def show(lt: list):
    print('列表排序中')
    lt.sort()
    print(lt)


if __name__ == '__main__':
    lt = [3213, 21.3, True, '+', 3.21, '-', 3.21, 3.21, 3.21, 3.21, 3213123, 321, 3, 21, 3, 21, 3, 21, 321]
    lt = [3213, 21.3, 3.21, 3.21, 3.21, 3.21, 3.21, 3213123, 321, 3, 21, 3, 21, 3, 21, 321]
    show(lt)

```

> 装饰器带参数和返回值

```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: gg.py
@time: 2021/1/29 3:47 下午
'''


def decorator(func):
    def inside(*args, **kwargs):
        return func(*args, **kwargs) + 10
    return inside


@decorator
def ping_fang(n):
    return n * n


if __name__ == '__main__':
    print(ping_fang(4))
```