---
title: Python魔法方法
categories:
  - python
tags:
  - python
abbrlink: c978b9d6
date: 2020-08-11 10:19:04
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->

# Python魔法方法


## super函数

super() 函数是用于调用父类(超类)的一个方法
Python 3 可以使用直接使用 `super().xxx` 代替 `super(Class, self).xxx`

## __init__初始化构造函数
在类创建的过程中，会先去执行`__init__`这个方法，用来初始化整个类中的内容
```python
class Person(object):

    def __init__(self, name, age):
        self.name = name
        self.age = age

p1 = Person('张三', 25)
p2 = Person('李四', 30)
```

## __new__真正的构造函数
其实在调用``__init__``之前，程序会先去调用``__new__``方法，这个方法我们一般很少定义，不过我们在一些开源框架中偶尔会遇到定义这个方法的类。实际上，这才是“真正的构造方法”，它会在对象实例化时第一个被调用，然后再调用``__init__``
``__new__``所做的一些事情

- ``__new__``的第一个参数是`cls`，而__init__的第一个参数是`self`
- ``__new__``返回值是一个实例，而``__init__``没有任何返回值，只做初始化操作
- ``__new__``由于是返回一个实例对象，所以它可以给所有实例进行统一的初始化操作
由于``__new__``优先于``__init__``调用，且返回一个实例，所以我们可以利用这种特性，每次返回同一个实例来实现一个单例类

```python
class Singleton(object):
    """单例"""
    _instance = None  # 用来记录类的实例是否被创建
    def __new__(cls, *args, **kwargs):  # 接收一个类的实例返回给__init__进行初始化
        if not cls._instance:
            cls._instance = super().__new__(cls, *args, **kwargs)
        return cls._instance

class MySingleton(Singleton):
    pass

a = MySingleton()
b = MySingleton()

assert a is b	# True
```

另外一种使用场景是当你需要继承内置类时，例如int、str、tuple，只能通过__new__来达到初始化数据的效果

```python
class Sign(float):
    def __new__(cls,kg):
        return float.__new__(cls,kg*2)


if __name__ == '__main__':
    s = Sign(29.0)
    print(s)  # res = 58.0
```

## __del__析构函数
这个函数主要是在对象被垃圾回收时被调用，但是并不会在`del x`的时候被调用，
由于Python是通过引用计数来进行垃圾回收的，也就是说，如果这个实例还是有被引用到，即使执行del销毁这个对象，但其引用计数还是大于0，所以不会触发执行``__del__``
```python
class Person(object):
    def __del__(self):
        print("omg ,is delete")

if __name__ == '__main__':
    p = Person()
# 被删除的时候会出现 omg is delete
```


## __str__ / __repr__函数

- `__str__`强调可读性，而`__repr__`强调准确性/标准性
- `__str__`的目标人群是用户，而`__repr__`的目标人群是机器，它的结果是可以被执行的
- %s调用`__str__`方法，而%r调用`__repr__`方法

```python
class Person(object):    
    def __str__(self):
        return "helloworld"
    def __repr__(self):
        return "helloworld1"
if __name__ == '__main__':
    p = Person()
    print(str(p)) # helloworld
    print("{}".format(p)) #  helloworld
    print("%s"%p) # helloworld
    print(repr(p))  # helloworld1
```

- 如果只定义了_str__，那么repr(person)输出<__main__.Person object at 0x1093642d0>
- 如果只定义了__repr__，那么str(person)与repr(person)结果是相同的


## __unicode__
```python
class Person(object):

    def __unicode__(self):
        # 这里不是u'hello'
        return 'hello'
    
person = Person()
print unicode(person)	# helllo
print type(unicode(person))	# <type 'unicode'>
```
不怎么常用


## __hash__ / __eq__
```python
class Person(object):
    def __init__(self, uid):
        self.uid = uid
        
	def __repr__(self):
        return 'Person(%s)' % self.uid
        
    def __hash__(self):
        return self.uid
    
    def __eq__(self, other):
        return self.uid == other.uid
    
p1 = Person(1)
p2 = Person(1)
p1 == p2	# True

p3 = Person(2)
print (set([p1, p2, p3]))	# 根据唯一标识去重输出 set([Person(1), Person(2)])
```
也就是在调用`"a==b"`的时候会去调用`__eq__`函数,然后那他们各自的属性进行比较