---
title: ^python内置函数
date: 2021-01-31 22:09:52
categories:  [python]
tags: [python]
---


<!--more-->


# ^python内置函数


## classmethod
classmethod 修饰符对应的函数不需要实例化，不需要 self 参数，但第一个参数需要是表示自身类的 cls 参数，可以来调用类的属性，类的方法，实例化对象等

```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: ii.py
@time: 2021/1/31 10:06 下午
'''


class Test(object):

    @classmethod
    def echo_message(cls, msg: str):
        print(msg)


if __name__ == '__main__':
    Test.echo_message("helloworld")

# res: helloworld
```


## staticmethod
python staticmethod 返回函数的静态方法。
该方法不强制要求传递参数，如下声明一个静态方法：
```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: ii.py
@time: 2021/1/31 10:06 下午
'''


class Test(object):

    @staticmethod
    def echo_age(age: int):
        print(age)


if __name__ == '__main__':
    Test().echo_age(12)
```

## isinstance
isinstance() 函数来判断一个对象是否是一个已知的类型，类似 type()。
```python
class Test(object):

    def __init__(self, age: int):
        self.age = age

    def echo_age(self):
        print(self.age)


if __name__ == '__main__':
    print(isinstance(Test, object))
    tt = Test(22)
    tt.echo_age()
    print(isinstance(tt,object))

```

## @property

```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: ii.py
@time: 2021/1/31 10:06 下午
'''


class Test(object):
    def __init__(self):
        self._score = 0

    def get_source(self) -> int:
        return self._score

    def set_source(self, value: int):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value


if __name__ == '__main__':
    tt = Test()
    tt.set_source(20)
    value = tt.get_source()
    print(value)
```

```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: ii.py
@time: 2021/1/31 10:06 下午
'''


class Test(object):
    def __init__(self):
        self._score = 0

    @property
    def source(self) -> int:
        return self._score

    @source.setter
    def source(self, value: int):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value


if __name__ == '__main__':
    tt = Test()
    tt.source = 20
    print(tt.source)


```

## __call__ 让类的实例当做函数来使用
```python
class Test(object):

    def __call__(self):
        print("is  __call__")


if __name__ == '__main__':
    tt = Test()
    tt()

# is  __call__

```

## __module__ , __class__ 

- __module__ 表示当前操作的对象在属于哪个模块。
- __class__ 表示当前操作的对象属于哪个类

```python
class Foo:
    pass

obj = Foo()
print(obj.__module__)
print(obj.__class__)

# 运行结果：
# __main__
# <class '__main__.Foo'>
```



## 继承
```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: ii.py
@time: 2021/1/31 10:06 下午
'''


class A(object):

    def echo_message(self) -> None:
        print("i am a")


class B(A):

    def echo_message2(self) -> None:
        print("i am b")
        super().echo_message()


if __name__ == '__main__':
    b = B()
    # i am b
    # i am a
    b.echo_message2()
    # i am a
    b.echo_message()

```

### 使用实例一
继承的时候保证父类被初始化
```python
class A(object):

    def __init__(self):
        self.a = "a message"

    def echo_message(self) -> None:
        print("i am a")


class B(A):
    def __init__(self):
        self.b = "b mseeage"

    def echo_message3(self) -> None:
        print(self.a)


if __name__ == '__main__':
    b = B()
    b.echo_message3() #  报错

```
这段代码中，报错的原因就是B虽然继承了A 但是没有在初始化的时候 初始化A
```python
class A(object):

    def __init__(self):
        self.a = "a message"

    def echo_message(self) -> None:
        print("i am a")


class B(A):
    def __init__(self):
        super().__init__() # 保证初始化父类
        self.b = "b mseeage"

    def echo_message3(self) -> None:
        print(self.a)


if __name__ == '__main__':
    b = B()
    b.echo_message3()

```
如上初始化以后就OK了

### 使用实例二
用来初始化自身，python中有__new__魔术方法，在生成类的时候，会先去调用它，然后再去调用__init__来初始化类的变量或者属性

```python
class A(object):

    def __init__(self, age: int, name: str):

        print("B")
        self.age = age
        self.name = name

    def __new__(cls, *args, **kwargs):
        print("A")
        return super(A, cls).__new__(cls)


if __name__ == '__main__':
    # res: A
    a = A(12, "hzj")

# res:
# A
# B
```
先使用__new__方法，然后再用super调用自身创建一个A的类，类再调用__init__ 实例化
代码中重构了父类的new方法，执行new的过程就是person类创建的过程，所以__new__方法实际上就是创建这个类实例方法。（这里指的是A类

> 当解释器解释到a = A(12,"hzj")时候，先执行__new__(cls,*args,**kwargs),并执行父类的__new__方法，将name，age参数传入父类__new__方法，创建A,注意A也适用父类的，A的父类是type，type是一个元类

__init__ 通常用于初始化一个新实例，控制这个初始化的过程，比如添加一些属性， 做一些额外的操作，发生在类实例被创建完以后。它是实例级别的方法
__new__ 通常用于控制生成一个类实例的过程。它是类级别的方法

### __new__的作用
```python
class myint(int):
    def __new__(cls, *args,**kwargs):
        print('exec new....')
        return super(myint,cls).__new__(cls,abs(*args))

print(myint(-1))#自定义int类
print(int(-1))#自带的int类
结果：
exec new....
1
-1
```

### __new__方法实现单例模式
```python
class Single(object):

    def __new__(cls):
        if not hasattr(cls,"test"):
            cls.test = super(Single,cls).__new__(cls)
        return cls.test


if __name__ == '__main__':
    s1 = Single()
    s2 = Single()
    print(s1.test)
    print(s2.test)
    print(s1.test is s2.test)
```

> 实现单例模式2

```python
class Single(object):
    _is_instance = None

    def __new__(cls, *args, **kwargs):
        if not cls._is_instance:
            cls._is_instance = super(Single,cls).__new__(cls)
        return cls._is_instance


if __name__ == '__main__':
    s1 = Single()
    s2 = Single()
    if s1 is s2:
        print("true")
```


## hasattr 用于判断对象是否包括对应的属性
```python
class Single(object):
    i = 2


if __name__ == '__main__':
    s1 = Single
    print(hasattr(s1,"i")) # True
```



## python 元编程
![2021-02-01-08-48-23](http://noback.upyun.com/2021-02-01-08-48-23.png)
元编程的最大目的之一就是将类的声明动态化，比如我们使用的orm，为了适配每一张表，必须要有对应的类才行，于是元类产生了。
以前我们创建类，常用这样的方式
```python
class Factory(object):
    pass
```
但事实上，这本身就已经是一个类的实例化过程了，在这之上还有一层也就是type
type的父类是type type的子类是object 而 objectj的子类就是我们平常所创建的类。如下
```python
class Factory(object):
    pass


if __name__ == '__main__':
    print(type(Factory)) # <class 'type'>
    fc = Factory()
    print(type(fc)) # <class '__main__.Factory'>

    print(isinstance(Factory,type)) # True
    print(isinstance(fc,Factory)) # True
```
![2021-02-01-08-41-46](http://noback.upyun.com/2021-02-01-08-41-46.png)

### 使用元类创建

根据上面的结论，我们可以用type来创建一个类,元类的初始化过程中，需要传入三个参数
- <name>指定类名称。这成为__name__该类的属性。
- <bases>指定从其继承的基类的元组。这成为__bases__该类的属性。
- <dict>指定一个包含类主体定义的名称空间字典。这成为__dict__该类的属性。

> 例子1

```python
>>> Foo = type('Foo', (), {})
>>> x = Foo()

>>> x.__class__
<class '__main__.Foo'>

>>> x.__class__.__name__
'Foo'

>>> x.__class__.__bases__
(<class 'object'>,)

>>> x.__class__.__dict__
mappingproxy({'__module__': '__main__', '__dict__': <attribute '__dict__' of 'Foo' objects>, '__weakref__': <attribute '__weakref__' of 'Foo' objects>, '__doc__': None})
```
```python
>>> class Foo:
...     pass
...
>>> x = Foo()

>>> x
<__main__.Foo object at 0x0370AD50>
```

> 例子2

```python
>>> Bar = type('Bar', (Foo,), dict(attr=100))

>>> x = Bar()

>>> x.attr
100

>>> x.__class__
<class '__main__.Bar'>

>>> x.__class__.__bases__
(<class '__main__.Foo'>,)
```
这里表示类的名字是Bar 继承的类来自Foo 自带的属性是attr=10
类似于这样
```python
>>> class Bar(Foo):
...     attr = 100
...

>>> x = Bar()

>>> x.attr
100

>>> x.__class__
<class '__main__.Bar'>

>>> x.__class__.__bases__
(<class '__main__.Foo'>,)
```

> 例子三 创建类函数

```python
>>> Foo = type(
...     'Foo',
...     (),
...     {
...         'attr': 100,
...         'attr_val': lambda x : x.attr
...     }
... )

>>> x = Foo()

>>> x.attr
100

>>> x.attr_val()
100
```

```python
>>> class Foo:
...     attr = 100
...     def attr_val(self):
...         return self.attr
...

>>> x = Foo()

>>> x.attr
100

>>> x.attr_val()
100
```

> 例子4

```python
>>> def f(obj):
...     print('attr =', obj.attr)
...
>>> Foo = type(
...     'Foo',
...     (),
...     {
...         'attr': 100,
...         'attr_val': f
...     }
... )

>>> x = Foo()
>>> x.attr
100
>>> x.attr_val()
attr = 100
```

```python
>>> def f(obj):
...     print('attr =', obj.attr)
...
>>> class Foo:
...     attr = 100
...     attr_val = f
...

>>> x = Foo()
>>> x.attr
100
>>> x.attr_val()
attr = 100
```

## 元类
元类是类的类。类定义类的实例（即对象）的行为，而元类定义类的行为。类是元类的实例。

metaclass创建orm
```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: ii.py
@time: 2021/1/31 10:06 下午
'''

import numbers
import string


# 字段类型
class Field(object):
    def __init__(self, min_lenth, max_lenth):
        self.min_lenth = min_lenth
        self.max_lenth = max_lenth
        if min_lenth >= max_lenth:
            raise ValueError("最小值应该比最大值小")


class IntField(Field):
    def __init__(self, min_lenth: int, max_lenth: int):
        self._value = None
        super(IntField, self).__init__(min_lenth, max_lenth)

    def __get__(self, instance, owner):
        return self._value

    def __set__(self, instance, value):
        if isinstance(value, numbers.Integral):
            if self.min_lenth:
                if value < self.min_lenth:
                    raise ValueError("必须大于等于最小值")
            if self.max_lenth:
                if value > self.max_lenth:
                    raise ValueError("必须小于等于最大值")
            self._value = value
        else:
            raise ValueError("参数必须输一个int类型")

    def __delete__(self, instance):
        pass


class CharField(Field):
    def __init__(self, min_lenth: int, max_lenth: int):
        self._value = None
        super(CharField, self).__init__(min_lenth, max_lenth)

    def __get__(self, instance, owner):
        return self._value

    def __set__(self, instance, value):
        if isinstance(value, string.ascii_uppercase):
            raise ValueError("必须全部小写")
        else:
            self._value = value

    def __delete__(self, instance):
        pass


class BaseModel(type):

    def __new__(cls, name, base, attrs, *args, **kwargs):

        fields = dict()
        for key, value in attrs.items():
            if isinstance(value, Field):
                fields[key] = value
        meta = attrs.get("Meta")
        db_name = name.lower()
        if meta:
            data = meta.__dict__
            if 'db_name' in data:
                db_name = data.get("db_table")
            attrs['data'] = data
            del attrs['Meta']

        attrs["db_table"] = db_name
        attrs["fields"] = fields

        return super().__new__(cls, name, base, attrs)


class Model(metaclass=BaseModel):
    def __init__(self, *args, **kwargs):
        for key, value in kwargs.items():
            if getattr(self, key, None):
                setattr(self, key, value)

    def save(self):
        fields = []
        values = []
        for key, value in self.fields.items():
            fields.append(key)
            values.append(str(getattr(self, key)))
        sql = "insert in into {db_table}({fields}) values{values}".format(db_table=self.db_table,
                                                                          fields=",".join(fields),
                                                                          values=",".join(values))
        print(sql)


class UserModel(Model):
    name = CharField(max_lenth=10, min_lenth=2)
    height = IntField(max_lenth=300, min_lenth=50)

    class Meta:
        db_table = "用户"


if __name__ == "__main__":
    # user = User()
    # user.name = "witt"
    # user.height = 170
    user = UserModel(name="witt", height=170)
    user.save()
```