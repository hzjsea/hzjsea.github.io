---
title: python 基础
categories:
  - python
tags:
  - python
abbrlink: 98b77d55
date: 2020-01-04 15:39:26
---
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [python基础](#python基础)
  - [python创建虚拟环境](#python创建虚拟环境)
  - [使用pipenv](#使用pipenv)
  - [使用__slot__](#使用__slot__)
    - [给类的特定实例绑定属性](#给类的特定实例绑定属性)
    - [给类的特定实例绑定方法](#给类的特定实例绑定方法)
    - [给类的所有实例绑定方法](#给类的所有实例绑定方法)
    - [限制实例的属性](#限制实例的属性)
    - [__slots__继承](#__slots__继承)
    - [使用@property](#使用property)

<!-- /code_chunk_output -->
<!-- more -->

# python基础


## python创建虚拟环境
- 方法1 使用virtualenv
```python
pip install virtualenv
# 豆瓣源
# pip install -i https://pypi.douban.com/simple virtualenv 

virtualenv env # 创建虚拟环境
virtualenv -p /usr/local/bin/python3 venv # 指定解释器安装
source env/bin/active # 激活虚拟环境
deactivate # 退出虚拟环境
rm -rf ./venv # 删除虚拟环境
```

## 使用pipenv
https://juejin.im/post/5ca4be8bf265da30b160de27


## 使用__slot__
python是一门动态语言，创建class的实例之后，我们可以给该实例绑定任何属性和方法

举例:

### 给类的特定实例绑定属性
```py
  class student(object):
      pass

  if __name__ == "__main__":
      # 尝试给实例绑定动态变量
      s = student()
      s.name = 'hzj'
      print(s.name)
```


### 给类的特定实例绑定方法
```py
class student(object):
    pass


def set_age(self,age):
    self.age = age

if __name__ == "__main__":
    s = student() # 创建实例
    from types import MethodType
    s.set_age = MethodType(set_age,s) # 绑定方法
    s.set_age(25)
    print(s.age)
```
但是给一个实例绑定的方法对另一个实例是不起作用的
```py
  class student(object):
      pass

  def set_age(self,age):
      self.age = age


  if __name__ == "__main__":
      s = student()
      from types import MethodType
      s.set_age = MethodType(set_age,s)
      s.set_age(25)

      s2 = student() # 新的实例
      s2.set_age(50)
      print(s2.age)

# 报错
# Traceback (most recent call last):
#   File "slots_py.py", line 15, in <module>
#     s2.set_age(50)
# AttributeError: 'student' object has no attribute 'set_age'
```

### 给类的所有实例绑定方法
```py
  class student():
      pass

  def set_age(self,age):
      self.age = age


  if __name__ == "__main__":
      student.set_age = set_age
      s = student()
      s.set_age(10)
      print(s.age)

      s2 = student()
      s2.set_age(20)
      print(s2.age)

```

### 限制实例的属性
上来的三例都是给实例扩展属性，在某些情况下我们可以用__slots__ = ('x','y') 的形式来定义绑定的属性名称
```py
  class student():
      __slots__ = ('name','age')

  if __name__ == "__main__":
      s1 = student() # 声明实例
      s1.age = 2
      print(s1.age)

      # 声明不允许的变量
      s2 = student()
      s2.hobit = "swimming"
      print(s2.hobit)
```
出现报错
```py

Traceback (most recent call last):
  File "slots_py.py", line 15, in <module>
    s2.hobit = "swimming"
AttributeError: 'student' object has no attribute 'hobit'
```

### __slots__继承
在子类当中，他是不具有父类的属性的，除非在子类中也定义__slots__，这样，子类实例允许定义的属性就是自身的__slots__加上父类的__slots__。

```py

  class student():
      __slots__ = ('name','age')


  class BadStudent(student):
      __slots__ = ('name','hobit')



  if __name__ == "__main__":
      bs = BadStudent()
      bs.name = "name"
      bs.hobit = "hobit"
      bs.age = "age"
      bs.xx = "xx"

      print(bs.name,bs.hobit,bs.age)
      print(bs.xx)

Traceback (most recent call last):
  File "slots_py.py", line 15, in <module>
    bs.xx = "xx"
AttributeError: 'BadStudent' object has no attribute 'xx'
```
### 使用@property
```py
# 使用property


## 在绑定属性时，如果我们直接把属性暴露出去，虽然写起来很简单，但是，没办法检查参数，导致可以把成绩随便改：

class student :
    __slots__ = ('name','age','score')



class student2:
    def get_score(self):
        return self._score

    def set_score(self,value):
        if not isinstance(value,int):
            raise ValueError("score must be an integet!")
        if vlaue < 0 or value > 100:
            raise ValueError("some must between 0 ~ 100!")
        self._score = value

class student3(object):
    @property
    def score(self):
        return self._score

    @score.setter
    def score(self,value):
        if not isinstance(value,int):
            raise ValueError("score must be an integet!")
        if vlaue < 0 or value > 100:
            raise ValueError("some must between 0 ~ 100!")
        self._score = value



if __name__ == "__main__":
    s = student()
    s.score = 10
    print(s.score)


    s2 = student2()
    s2.set_score(20)
    print(s2.get_score())


    s3 = student3()
    s3.score = 30 # @property =  set_score
    print(s3.score) #@score.setter = get_score


## 只用了property的话就是只读属性
## 用了property和setter的话就是可读写属性

```