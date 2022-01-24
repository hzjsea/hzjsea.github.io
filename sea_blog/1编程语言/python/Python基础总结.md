## python基础总结
## 聊一聊Python中的类
什么是Python的类，很多人肯定会随手打出
```python
class Student(object):
	def __init__(self):
		pass
jack = Student()
Helen = Student()
```
完事了，但其实不然，Python的类还是有很多值得考虑和记录的地方

#### 基础向
如上所说，我们先来理一理Python类的基础，首先既然说到类，我们就离不开对象这个概念，Python是一门解释型动态语言，在Python中处处皆对象。既然有了对象，我们当然要具有生产这些对象的模板，我们把它称为类。在Python中，Class简称是抽象类的统称，比如上方的Student就是一个抽象的概念，他是一个模板。那么jack和Helen则是他的实例。于是面向对象中，类(Class)和实例(Instance)的概念就出来了，记住类只是一个抽象的概念，是模板。而实例则是由这些模板生产出来的实例。

#### 继承最基础的类
看到现在我们应该有一个概念，也就是说我们创造一个类，我们可以使用他创造出多个实例。那么多个类，就可以创造出多个多个实例。那么其实这些类都继承自一个最基础的类object，在Python2.4-3中根据是否继承object类来区分新式类和经典类，但是在python3中，已经默认继承Object类了,所以你完全可以这样写
```python
class Student:
	def __init__(self):
		pass
```

#### 类的内存与实例
既然类是创建实例的模板，那么必须有存放类的内存的地方
```python
class Student(object):
	def __init__(self):
		pass

if __name__ == "__main__":
	print(Student)
	print(Student())
    a = Student()
    print(a)
```
输出的结果如下
```text
PS E:\app> & C:/Users/alpaca/AppData/Local/Programs/Python/Python37/python.exe e:/app/demo1.py
<class '__main__.st'>
__init__
<__main__.st object at 0x000001BB998EFB88>
__init__
<__main__.st object at 0x000001BB998EFB88>
```
输出的流程
```text
# st返回的是  <class '__main__.st'>  类的对象名
# print(st())时 st()创建了一个实例，python会为他创建一个实例，赋值给st() 然后再由print输出，因此会先执行__init__初始化,即先输出"__init__",再输出类的实例
# a = st() 创建一个实例a,会执行__init__ 后再输出 类的实例 
# 输出类的实例内容包括  <__main__.st     object at 0x00000141C826FE08>  创建实例的类的对象名 和 实例的存储地址
```

#### 类的属性和方法
既然都说了处处皆对象，那么对象也可以是千变万化的，正如jack和helen也一定存在着不同的地方，这些他们所具有的相同或不同的内容我们称之为**属性**，为了方便我们将固有属性都放在类的中，使用__init__在构造函数的初始化阶段定义属性
```Python
class Student(object):
    def __init__(self,name,age):
        # 因人而异的属性
        self.name = name
        self.age = age

        # 固有的属性
        self.type = "People"


if __name__ == "__main__":
    jack = Student("jack",12)
    Helen = Student("Helen",18)

    print(jack.name)
    print(Helen.name)
	
    # 输出固有属性
    print(jack.type)
    print(Helen.type)

    # 随着年龄的增增长，属性也会随之变大
    Helen.age = Helen.age + 1
    jack.age +=1
    print(jack.age)
    print(Helen.age)

    # Helen比较自私，他有别人没有的属性
    Helen.HasCar = "Helen has one car "
    print(Helen.HasCar)
```
输出内容
```Text
PS E:\app> & C:/Users/alpaca/AppData/Local/Programs/Python/Python37/python.exe e:/app/demo2.py
jack
Helen
People
People
13
19
Helen has one car
```
记住，实例与实例之间的属性是相互独立的。

**私有属性**
Helen是一个女孩子，他很珍惜她的年龄，因此他不希望自己的年龄是会增大的，于是他想要将年龄这个属性据为己有，变成私有的属性。于是他在age的开头加上了双下划线\_\_ 变成了\_\_age，私有的(private)，当你在外部想要去改变的时候，你会发现报错了。
```python
class Student(object):
    def __init__(self,name,age):
        # 因人而异的属性
        self.name = name
        self.__age = age

if __name__ == "__main__":
    jack = Student("jack",12)
    Helen = Student("Helen",18)

    print(jack.age)
    jack.age +=1
    print(jack.age)
```
输出的结果
```Text
PS E:\app> & C:/Users/alpaca/AppData/Local/Programs/Python/Python37/python.exe e:/app/3.py
Traceback (most recent call last):
  File "e:/app/3.py", line 17, in <module>
    print(jack.age)
AttributeError: 'Student' object has no attribute 'age'
```
其实加了双下划线也不是一定不能访问，我们可以使用\_Student\_\_age去获取
当然年龄这些东西他是改变不了的，我们想办法去得到这些，于是我们要添加对应的方法
```
class Student(object):
    def __init__(self,name,age):
        # 因人而异的属性
        self.name = name
        self.__age = age
		
    def get_age(self):
        return self.__age

    def __set_age(self,age):
        self.__age = age

if __name__ == "__main__":
    jack = Student("jack",12)
    Helen = Student("Helen",18)

    print(Helen.get_age())
    Helen._Student__set_age(19)
    print(Helen.get_age())

```


除了属性之外，类还可以包含方法，同样的我们通用(固有)的类的方法也会放进类中
```bash
class Student(object):
    def __init__(self,name,age):
        # 因人而异的属性
        self.name = name
        self.age = age

    def say(self,msg):
        print(msg+", my name is "+ self.name)

if __name__ == "__main__":
    jack = Student("jack",12)
    Helen = Student("Helen",18)

    jack.say("hello")
```
输出内容为
```bash
PS E:\app> & C:/Users/alpaca/AppData/Local/Programs/Python/Python37/python.exe e:/app/demo4.py
hello, my name is jack
```
类里面的所有方法都是以self为第一个参数，就指向创建的类本身，在调用时不需要传入任何的值


#### 类的继承
在上面我们讲到过 所有的类都继承自object这个基础类。同样的，在类与类之间也可以互相继承，有了这个，在之前我们实现过的helen拥有汽车的事就不用遮遮掩掩的了，他完全可以自己再继承一个类
```bash
class Student(object):
    def __init__(self,name,age):
        # 因人而异的属性
        self.name = name
        self.age = age
		
        # 固定的属性
        self.type = "People"

class Man_HasCar(Student):
    def HasCar(self):
        print(self.name+" has one car ")

if __name__ == "__main__":
    jack = Student("jack",12)
    Helen = Man_HasCar("Helen","18")

    Helen.HasCar()
```


#### 获取对象的类型
**type()**
使用type()来判断对象类型
**isinstance()**
使用isinstance来判断某一个实例是否是某个类的实例，层级迭代也算，即假设B的实例是b,B的父类是A，print(instance(b,A)) = true
基本类型也可以用isinstance来判断 isinstance('a',str)
```python
print(instance(b,A)) # true
isinstance('a',str) # True
isinstance([1, 2, 3], (list, tuple)) # True
```
**dir()**
如果要获取一个对象的所有属性和方法，可以使用dir函数,他返回一个包含字符串的list,比如获取一个str对象的方法
```python
dir("ABC")
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isascii', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
```
再这之中含双下划线的都是一些魔法方法，比如返回字符的长度，当然我们也可以使用len()去获取，但实际仍然就是调用了\_\_len\_\_这个方法
```python
print("ABC".__len__())
print(len("ABC"))

class return_len_string(object):
    # 重写__len__
    def __len__(self):
        return 100

rls = return_len_string()
print(len(rls))
# 返回100
```

#### 获取对象属性
python准备了getattr()、setattr()、hasattr()，可以直接操作一个对象的属性
```python
# 获得属性
getattr(cls,'name')
# 设置属性
setattr(cls,'name','Helen')
# 判断某个类是否具有某个属性
hasattr(cls,'name')
```

#### 特殊的属性\_\_slot\_\_
我们知道由于Python是动态类的语言，再我们创建完一个类之后，并创建了他的实例，那么这个实例可以添加和绑定任何属性和方法，如果我们要限制实例的属性该怎么办，为了达到限制的目的，Python允许在定义class的时候，定义一个特殊的__slots__变量，来限制该class实例能添加的属性
```python
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称

"""实际执行效果"""
>>> class Student(object):
...     __slots__ = ('name', 'age')
...
>>> s = Student()
>>> s.name = 'digg'
>>> s.age = '19'
>>> s.score = 99
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'score'
>>>

```
**另外\_\_slot\_\_只对当前类有效，对继承类无效**



#### \_\_str\_\_ ，\_\_repr\_\_
该魔法方法自定义返回内容
```python
class tt(object):
    def __str__(self):
        return "自定义"
	
	def __repr__(self):
		return "自定义"
	# 或者
	__repr__ = __str__
class ss(object):
    pass

if __name__ == "__main__":
    print(tt())
    print(ss())
```
\_\_repr\_\_其实和\_\_str\_\_差不多,大多数情况下使用str就差不多了

#### \_\_iter\_\_ 和 \_\_next\_\_
如果一个类想被用于for ... in循环，类似list或tuple那样，就必须实现一个__iter__()方法，该方法返回一个迭代对象，然后，Python的for循环就会不断调用该迭代对象的__next__()方法拿到循环的下一个值，直到遇到StopIteration错误时退出循环。
```python
class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1 # 初始化两个计数器
    def __iter__(self):
        return self # 实例本身即是迭代对象，故而返回自己
    def __next__(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a > 100000: # 退出循环条件
            raise StopIteration();
        return self.a
# 测试
for n in Fib():
    print(n)
```

#### 枚举类

#### 元类


------------


## staticmethod classmethod使用方法与区别

一般来说，要使用某个类的方法，需要先实例化一个对象再调用方法。
而使用@staticmethod或@classmethod，就可以不需要实例化，直接类名.方法名()来调用。
这有利于组织代码，把某些应该属于某个类的函数给放到那个类里去，同时有利于命名空间的整洁。
既然@staticmethod和@classmethod都可以直接类名.方法名()来调用，那他们有什么区别呢
从它们的使用上来看,
- @staticmethod不需要表示自身对象的self和自身类的cls参数，就跟使用函数一样。
- @classmethod也不需要self参数，但第一个参数需要是表示自身类的cls参数。
- @classmethod 是一个函数修饰符，它表示接下来的是一个类方法，而对于平常我们见到的则叫做实例方法。
- 类方法的第一个参数cls，而实例方法的第一个参数是self，表示该类的一个实例。 
- 普通对象方法至少需要一个self参数，代表类对象实例
类方法有类变量cls传入，从而可以用cls做一些相关的处理。并且有子类继承时，调用该类方法时，传入的类变量cls是子类，而非父类。
- 对于类方法，可以通过类来调用，就像Test.foo()，有点类似C＋＋中的静态方法, 也可以通过类的一个实例来调用，就像Test().foo()，这里Test()，写成这样之后它就是类的一个实例了。   
- 静态方法则没有，它基本上跟一个全局函数相同，一般来说用的很少
如果在@staticmethod中要调用到这个类的一些属性方法，只能直接类名.属性名或类名.方法名。
而@classmethod因为持有cls参数(当然，也可以用“self” 代替，个人认为是为了和类的self区分才用cls的)，可以来调用类的属性，类的方法，实例化对象等，避免硬编码。

## 例子
```python
class cs(object):
    def speak(self,name):
        print("can you speak english? %s"%name)
		
    @staticmethod
    def say(name):
        print("my name is %s "%(name))

    @classmethod
    def run(cls,name):
        print("running now, %s"%(name))

if __name__ == "__main__":    
    cs.say("hzj")
    cs.run("hzj")
```
-----------------

## 聊一聊Python中的装饰器
https://blog.csdn.net/zhouchen1998/article/details/82933893
<font color='red'>装饰器就是把几个函数共同的功能放在一起</font>

Python 拥有丰富强大的功能和表达特性，其中之一便是装饰器，装饰器能够在不改变函数、方法、类本身的情况下丰富他们的功能。
**应用场景**:比如QQ会员吧，对于普通的会员我们使用的是func，但是对于VIP会员则是func_vip的方法，但是func与func_vip大部分功能的重叠性很高，没有必要重写一个func_vip方法，只需要丰富一下func就可以使用了，这时候我们就可以使用装饰器


#### 前提
1.函数可以作为变量
```p？yton
def func(name):
    print("my name is %s"%name)

func_other = func()
func_other("hzj")
```
2.将函数传递给函数
```Python
def func1(name):
    return name
def func2(func):
    print(func)
func2(func1("hzj"))
```
3.无参函数嵌套函数
```python
def func_wrap():
    def prt_func():
        return 'hello,world'
    return prt_func

hlowld = func_wrap()
print(hlowld())
```
4.有参函数嵌套函数
```python
def func_wrap():
    def prt_func(name):
        return 'hello,'+name   
    return prt_func

hlo = func_wrap()
print(hlo('crossin'))
```
5.函数参数嵌套函数
```python
def func(name):
    return "hello %s"%(name)

def top_func(func):
    def sec_func(name):
        return "hello %s"%(func(name))
    return sec_func

tt = top_func(func)
print(tt("hzj"))
```

#### 讲一下装饰器
其实装饰器就是一种将函数当做参数的使用方法，具体如何使用我们来看一下，首先我们来看一下最普通的函数
```Python
# 最简单的函数
def general():
    return "i am general func"

if __name__ == "__main__":
	print(general())
# 带参数的最简单的参数
def general_params(name):
    return "i am general %s"%name
if __name__ == "__main__":
    print(general())
    print(general_params("func"))
```
接下来是**函数传递**，在python中参数可以作为变量在函数与函数之间传递
```python
# 函数作为变量
def bottom():
    return "i ma bottom func"
b = bottom
print(b())
# -----------------
# 函数之间传递
# 函数作为变量传递
def bottom():
    return "i am bottom func"

def top(func):
    return "%s,ok!i know!"%(func())

t = top(bottom)
print(t)

```
接下来是**函数的嵌套**
```Python
# 不带参嵌套
def outer():
    def inner():
        return "ok!"
    return inner
out = outer()
print(out())
# 带参嵌套
def outer(name):
    def inner(name):
        return "ok!%s"%name
    return inner
out = outer("a")
print(out("b"))
# 结果ok!b
```

----------------------------
### 装饰器
```python
# 首先定义一个普通的函数
def print_text(name):
    return 'hello,'+ name
# 再定义一个嵌套函数，分别以函数和普通的字符串作为参数
def add_tag(func):
    def prt_func(name):
        return '<p>{0}</p>'.format(func(name))    
    return prt_func
    
# 将函数作为参数传递给 add_tag
hlo = add_tag(print_text)
# 将 'crossin' 作为参数传递给 hlo
print(hlo('crossin'))
# 结果 : <p>hello,crossin</p>
```
可以写成这样
```python
def add_tag(func):
    def prt_func(name):
        return '<p>{0}</p>'.format(func(name))    
    return prt_func

@add_tag
def print_text(name):
    return 'hello,'+ name

print_text('crossin')
```
![](http://noback.top/usr/uploads/2019/09/498165289.png)



## 来源
在Python这门语言中，存在着很多的魔法方法，除了这篇文章所介绍的__new__ __init__ __call__以外还存在着其他的魔法方法

任何事物都有一个从创建，被使用，再到消亡的过程，在程序语言面向对象编程模型中，对象也有相似的命运：创建、初始化、使用、垃圾回收，不同的阶段由不同的方法（角色）负责执行。
定义一个类时，大家用得最多的就是 __init__ 方法，而 __new__ 和 __call__ 使用得比较少，这篇文章试图帮助大家把这3个方法的正确使用方式和应用场景分别解释一下。
关于 Python 新式类和老式类在这篇文章不做过多讨论，因为老式类是 Python2 中的概念，现在基本没人再会去用老式类，老式类必须显示地继承 object，而 Python3 中，只有新式类，默认继承了 object，无需显示指定，本文代码都是基于 Python3 来讨论。

## __init__方法
__init__方法负责对象的初始化,既然是面向对象的语言，存在对象，当然也存在对象的属性，而init就是用来初始化对象属性的模仿方法，时间为系统执行该方法前，其实该对象已经存在了，要不然初始化什么东西呢？先看例子
```bash
class A:
    def __init__(self, *args, **kwargs):
        print("__init__")
        super(A,self).__init__()
    def __new__(cls):
        print("__new__")
		print(cls)
        return super(A,cls).__new__(cls)
if __name__ == "__main__":
    A()

# -----------------
# result:
__new__
<class '__main__.A'>
__init__
```
也就是说__new__比__init__先执行，大致的执行流程为__new__接受一个cls,即当前类的并生成一个实例给init中的self,再由init初始化结构，另外__init__中除了self外，其他参数个数要与__new__相同
```bash
class B:
    def __init__(self, *args, **kwargs):
        print("init", args, kwargs)

    def __new__(cls, *args, **kwargs):
        print("new", args, kwargs)
        print(cls)
        self = super().__new__(cls)
        print(self)
        return super().__new__(cls)
if __name__=="main":
	B()
```

## __new__方法
其实在上面已经大致的能够知道__new__的用处了，即在初始化机构之前，返回一个类的对象实例给__init__.
一般我们不会去重写该方法，除非你确切知道怎么做，什么时候你会去关心它呢，它作为构造函数用于创建对象，是一个工厂函数，专用于生产实例对象。著名的设计模式之一，单例模式，就可以通过此方法来实现。
<font color=red>单例模式,在当前博客的图书馆一栏，设计模式中有讲到</font>

## __call__方法
__call__其实与callback回调函数有关，也就是可回调对象，说简单一点，该魔法方法是在当前类被调用时触发的
```bash
class C:
    def __init__(self):
        print("__init__")
    def __call__(self):
        print("__call__")

if __name__=="main":
	c = C()
	c()

# -----------
# result: 
__init__
__call__
```

