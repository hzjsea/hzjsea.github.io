---
title: ProPython3
date: 2021-02-16 18:57:52
categories:  [python]
tags: [python]
---


<!--more-->


# ProPython3

- cookbook https://python3-cookbook.readthedocs.io/zh_CN/latest/c04/p05_iterating_in_reverse.html
- ProPython3 url


## 第二章节

> Like any other book on programming, the remainder of this book relies on quite a few features that may or may not be considered commonplace by readers. You, the reader, are expected to know a good deal about Python and programming in general, but there are a variety of lesser-used features that are extremely useful in the operations of many techniques shown throughout the book.Therefore, as unusual as it may seem, this chapter focuses on a concept of advanced basics. The tools and techniques in this chapter aren’t necessarily common knowledge, but they form a solid foundation for more advanced implementations to follow. Let’s start off with some of the general concepts that tend to come up often in Python development


像其他任何的有关编程的书一样，这本书的其余部分依赖于很多平常不会读者注意到的功能。作为读者，您应该大致了解Python和编程的知识，这本书里有很多各种各样的较少使用，但非常有用的操作。因此，本章着眼于高级基础概念，虽然看起来很不寻常。本章中的工具和技术不一定是常识，但它们为后续更高级的实现奠定了坚实的基础,让我们从Python开发中经常出现的一些一般概念开始

> General  Concepts 
Before getting into more concrete details, it’s important to get a feel for the concepts that lurk behind the specifics covered later in this chapter. These are different from the principles and philosophies discussed in Chapter 1 in that they are concerned more with actual programming techniques, whereas those discussed previously are more generic design goals.Think of Chapter 1 as a design guide, whereas the concepts presented in this chapter are more of an implementation guide. Of course, there’s only so specific a description like this can get without getting bogged down in too many details, so this section will defer to chapters throughout the rest of the book for more detailed information.

基本概念
在开始更具体的细节之前，重要的是要了解潜伏在本章后面的特定细节概念。这些与第1章中讨论的原理和哲学不同之处在于它们更多地关注实际的编程技术，而先前讨论的是更通用的设计目标。以第1章为设计指南，而本章介绍的概念是更多实施指南。当然，在没有太多细节困扰的情况下，可以获得如此具体的描述，因此本节将参考本书其余各章的内容，以获取更多详细信息。

> Iteration(迭代)
Although there is a nearly infinite number of different types of sequences that might come up in Python code—more on that later in this chapter and in Chapter 5—most code that uses them can be placed in one of two categories: those that actually use the sequence as a whole and those that just need the items within it. Most functions use both approaches in various ways, but the distinction is important in order to understand what tools Python makes available and how they should be used.Looking at things from a purely object-oriented perspective, as opposed to a functional programming perspective, it’s easy to understand how to work with sequences that your code actually needs to use. You’ll have a concrete item such as a list, set, or dictionary, which not only has data associated with it but also has methods that allow for accessing and modifying that data. You may need to iterate over it multiple times, access individual items out of order, or return it from other methods for other code to use, all of which works well with more traditional object usage.Then again, you may not actually need to work with the entire sequence as a whole; you may be interested solely in each item within it. This is often the case when looping over a range of numbers, for instance, because what’s important is having each number available within the loop, not having the whole list of numbers available.The difference between the two approaches is primarily about intention, but there are technological implications as well. Not all sequences need to be loaded into memory, and many don’t even need to have a finite upper limit at all, such as a network stream. This category includes the set of positive odd numbers, squares of integers, and the Fibonacci sequence, all of which are infinite in length and easily computable. Therefore, they’re best suited for pure iteration, without the need to populate a list in advance, which also saves a bit of memory.The main benefit to this is memory allocation. A program designed to print out the entire range of the Fibonacci sequence only needs to keep a few variables in memory at any given time, because each value in the sequence can be calculated from the two previous values. Populating a list of the values, even with a limited length, requires loading all the included values into memory before iterating over them. If the full list will never be acted on as a whole, it’s far more efficient to simply generate each item as it’s necessary and discard it once it’s no longer required in order to produce new items.Python as a language offers a few different ways of implementation to iterate over a sequence without pushing all its values into memory at once. In its standard library, Python uses those techniques in many of its provided features, which may sometimes lead to confusion. Python allows you to write a for loop without a problem, but many sequences won’t have the methods and attributes you might expect to see on a list. To see two types of looping in action, try the following,
The section on iteration later in this chapter covers some of the more common ways to create iterable sequences and also a simple way to convert those sequences to lists when you truly do need to operate on the sequence as a whole. Sometimes, however, it’s useful to have an object that can function in both respects, which requires the use of caching


尽管Python代码中可能会出现几乎无限数量的不同类型的序列（在本章后面和第5章中会进行更多介绍），但大多数使用它们的代码可以归为以下两类之一：整个序列以及只需要其中的项目的序列。大多数功能使用两种方法都有不同的方式，但是区别对于理解Python提供了哪些工具以及应如何使用它们非常重要。从纯粹的面向对象的角度（而不是从函数编程的角度）来看事情，这很容易理解如何处理代码实际需要使用的序列。您将拥有一个具体的项目，例如列表，集合或字典，该项目不仅具有与之关联的数据，而且具有允许访问和修改该数据的方法。您可能需要多次迭代，无序访问单个项目或从其他方法返回它以供其他代码使用，所有这些都可以在更传统的对象用法上很好地工作。然后，您实际上可能不需要工作以整个序列为整体；您可能只对其中的每个项目感兴趣。例如，当遍历一个数字范围时通常是这种情况，因为重要的是要在循环中获得每个数字，而不是要获得整个数字列表。这两种方法之间的差异主要在于意图，但是也是技术方面的含义。并非所有序列都需要加载到内存中，许多序列甚至根本不需要具有有限的上限，例如网络流。此类别包括一组正奇数，整数平方和斐波那契数列，它们的长度都是无限的，并且易于计算。因此，它们最适合纯迭代，而无需事先填充列表，这也节省了一些内存。这样做的主要好处是内存分配。设计用于打印斐波纳契数列整个范围的程序在任何给定时间都只需要在内存中保留一些变量，因为该数列中的每个值都可以从前两个值计算得出。填充值的列表（即使长度有限）也需要在迭代之前将所有包含的值加载到内存中。如果完整列表永远不会付诸实践，那么根据需要简单地生成每个项目并在不再需要以生成新项目时将其丢弃就更有效了.Python作为一种语言提供了几种不同的处理方式实现迭代一个序列而不将其所有值立即推送到内存中。在其标准库中，Python在其提供的许多功能中使用了这些技术，有时可能会引起混淆。 Python使您可以毫无问题地编写for循环，但是许多序列都没有您期望在列表中看到的方法和属性。要查看两种类型的循环，请尝试以下操作
本章后面的“迭代”部分介绍了一些更常见的创建可迭代序列的方法，以及一种在确实需要对整个序列进行操作时将这些序列转换为列表的简单方法。但是，有时候，拥有一个可以在两个方面都起作用的对象很有用，这需要使用缓存

```python
#!/usr/bin/env python3
# encoding: utf-8
"""
@author: hzjsea
@file: iteration.py
@time: 2021/2/19 9:21 上午
"""


last_name = "Smith"
count = 0
for letter in last_name:
    print(letter, ' ', count)  # note a space between ' '
    count += 1


print('---and the second loop----')
count = 0
while count < 5:
    print(last_name[count], ' ', count)
    count += 1

# res
# S   0
# m   1
# i   2
# t   3
# h   4
# ---and the second loop----
# S   0
# m   1
# i   2
# t   3
# h   4
```

> caching(缓存)
Outside of computing, a cache is a hidden collection, typically of items either too dangerous or too valuable to be made directly accessible. The definition in computing is related, with caches storing data in a way that doesn’t impact a public-facing interface. Perhaps the most common real-world example is a Web browser, which downloads a document from the Web when it’s first requested but keeps a copy of that document. When the user requests that same document again (if the document has not changed) at a later time, the browser loads the private copy and displays it to the user instead of hitting the remote server again.
In the browser example, the public interface could be the address bar, an entry in the user’s favorites or a link from another web site, where the user never has to indicate whether the document should be retrieved remotely or accessed from a local cache. Instead, the software uses the cache to reduce the number of remote requests that need to be made, as long as the document doesn’t change quickly. The details of Web document caching are beyond the scope of this book, but it’s a good example of how caching works in general:
More specifically, a cache should be looked at as a time-saving or performance-boosting utility that doesn’t explicitly need to exist in order for a feature to work properly. If the cache gets deleted or is otherwise unavailable the function that utilizes it should continue to work properly, perhaps with a dip in performance because it needs to refetch the items that were lost. That also means that code utilizing a cache must always accept enough information to generate a valid result without the use of the cache.The nature of caching also means that you need to be careful about ensuring that the cache is as up-to-date as your needs demand. In the Web browser example, servers can specify how long a browser should hold on to a cached copy of a document before requesting a fresh one from the server. In simple mathematical examples, the result can be cached theoretically forever, because the result should always be the same, given the same input. Chapter 3 covers a technique called memoization that does exactly that.A useful compromise is to cache a value indefinitely but update it immediately when the value is updated. This isn’t always an option, particularly if values are retrieved from an external source, but when the value is updated within your application, updating the cache is an easy step to include, which saves the trouble of having to invalidate the cache and retrieve the value from scratch later on. Doing so can incur a performance penalty, however, so you’ll have to weigh the merits of live updates against the time you might lose by doing so.

在计算之外，缓存是一个隐藏的集合，通常是太危险或太有价值而无法直接访问的项目。计算中的定义与此相关，缓存以不影响面向公众的界面的方式存储数据。现实世界中最常见的示例可能是Web浏览器，该浏览器会在首次请求时从Web下载文档，但会保留该文档的副本。当用户在以后再次请求同一文档（如果文档未更改）时，浏览器将加载私有副本并将其显示给用户，而不是再次访问远程服务器
在浏览器示例中，公共界面可以是地址栏，用户收藏夹中的条目或来自其他网站的链接，用户无需在此指示是应远程检索文档还是应从本地缓存访问文档。取而代之的是，只要文档不会快速更改，该软件就会使用缓存来减少需要发出的远程请求的数量。 Web信息
文档缓存超出了本书的范围，但是它是一个很好的示例，说明了缓存一般如何工作
更具体地说，应该将缓存视为省时或提高性能的实用工具，为了使功能正常工作，该实用工具不需要明确存在。如果缓存被删除或由于其他原因不可用，则使用缓存的功能应继续正常运行，性能可能有所下降，因为它需要重新获取丢失的项目。这也意味着利用缓存的代码必须始终接受足够的信息以产生有效的结果而不使用缓存。缓存的性质还意味着您需要注意确保缓存与最新的一样。您的需求需求。在Web浏览器示例中，服务器可以指定浏览器在向服务器请求新文档之前，应将文档的缓存副本保留多长时间。在简单的数学示例中，由于在给定相同的输入的情况下结果应始终相同，因此该结果在理论上可以永久存储。第3章介绍了一种称为备忘录的技术，它可以做到这一点。一个有用的折衷办法是无限期地缓存一个值，但是在值更新时立即对其进行更新。这并不总是一种选择，特别是如果从外部源检索值时，但是当在应用程序中更新值时，更新缓存是包括在内的一个简单步骤，从而省去了使缓存无效并检索的麻烦。从头开始的价值。但是，这样做可能会导致性能下降，因此您必须权衡实时更新的优缺点与这样做可能会浪费的时间。

```python
import webbrowser
webbrowser.open_new('http://www.python.org/')
webbrowser.open_new_tab("https://www.baidu.com") 
```
这个库的操作就是打开一个网页


> Transparency
Whether describing building materials, image formats, or government actions, transparency refers to the ability to see through or inside of something, and its use in programming is no different. For our purposes, transparency refers to the ability of your code to see—and, in many cases, even edit—nearly everything that the computer has access to.Chapter 2 advanCed BasiCs
Python doesn’t support the notion of private variables typical in many other programming languages, so all attributes are accessible to any requester. Some languages consider that type of openness to be a risk to maintainability, instead allowing the code that implements an object to be solely responsible for that object’s data. Although that does prevent some occasional misuses of internal data structures, Python doesn’t take any measures to restrict access to that data
Although the most obvious use of transparent access is in class instance attributes—which is where many other languages allow more privacy—Python allows you to inspect a wide range of aspects of objects and the code that implements them. In fact, you can even get access to the compiled bytecode that Python uses to execute functions. Here are just a few examples of information available at runtime:•Attributes of an object
•The names of attributes available for an object
•The type of an object
•The module in which a class or function was defined•The location (typically a filename) in which a module was loaded
•The bytecode of a function object
Most of this information is only used internally, but it’s available because there are potential uses that can’t be accounted for when code is first written. Accessing or examining that information at runtime is called introspection and is a common tactic in systems that implement principles such as DRY (Don’t Repeat Yourself ). The definition by Hunt and Thomas for DRY is that “Every piece of knowledge must have a single, unambiguous, authoritative representation within a system” (The Pragmatic Programmer, 2000, by A. Hunt and D. Thomas).
The rest of this book contains many different introspection techniques in the sections where such information is available. For those rare occasions where data should indeed be protected, Chapters 3 and 4 show how data can show the intent of privacy or be hidden entirely.Chapter 2 advanCed BasiCs

> Control  Flow
Generally speaking, the control flow of a program is the path the program takes during execution. The more common examples of control flow, including of course the sequencestructure, are the if, for, and while blocks, which are used to manage the most fundamental branches your code could need. Those blocks are also some of the first things a Python programmer will learn, so this section will instead focus on some of the lesser-used and underutilized control flow mechanisms.

> Catching  Exceptions(捕捉异常)
Chapter 1 explained how Python philosophy encourages the use of exceptions wherever an expectation is violated, but expectations often vary between different uses. This is especially common when one application or module relies on another, but it’s also quite common within a single application. Essentially, any time one function calls another, it can add its own expectations on top of the exceptions the called function already handles.
Exceptions are raised with a simple syntax using the raise keyword, but catching them is slightly more complicated because it uses a combination of keywords. The trykeyword begins a block where you feel exceptions are likely to occur, while the exceptkeyword marks a block to execute when an exception is raised. The first part is easy, as try doesn’t have anything to go along with it, and the simplest form of except also doesn’t require any additional information:

第1章介绍了Python哲学如何鼓励在违反期望的地方使用异常，但是期望在不同的用法之间常常会有所不同。当一个应用程序或模块依赖于另一个应用程序或模块时，这尤其常见，但在单个应用程序中也很常见。从本质上讲，任何时候一个函数调用另一个函数时，它都可以在被调用函数已经处理的异常之上添加自己的期望.Exception使用简单的语法使用raise关键字引发异常，但是捕获它们则稍微复杂一点，因为它使用了组合关键字。 try关键字在您可能会感到异常的地方开始一个块，而exception关键字则标记了在引发异常时要执行的块。第一部分很简单，因为try不需要任何内容​​，而最简单的形式except则不需要任何其他信息

```python
def count_lines(file_name: str) -> int:
    try:
        return len(open(file_name, "r").readlines())
    except:
        print('exception error reading the file or calculating lines!')
        return 0


my_file = input("Enter file to open:")
print(count_lines(my_file))
```

Any time an exception is raised inside the try block, the code in the except block will be executed. As it stands, this doesn’t make any distinction among the many various exceptions that could be raised; no matter what happens, the function will always return a number. It’s actually fairly rare that you’d want to do that, however, because many exceptions should in fact propagate up to the caller—errors should never pass silently. Some notable examples are SystemExit and KeyboardInterrupt, both of which should usually cause the program to stop running.In order to account for those and other exceptions that your code shouldn’t interfere with, the except keyword can accept one or more exception types that should be caught explicitly. Any others will simply be raised as if you didn’t have a try block at all. This focuses the except block on just those situations that should definitely be handled, so your code only has to deal with what it’s supposed to manage. Make the minor changes to what you just tried, as shown here, to see this in action

每当try块内引发异常时，将执行except块中的代码。就目前情况而言，这在可能引发的多种异常之间没有任何区别；无论发生什么情况，该函数将始终返回一个数字。但是，实际上很少要这样做，因为实际上许多异常应该传播到调用方，因此错误永远都不应静默传递。一些值得注意的例子是SystemExit和KeyboardInterrupt，它们通常都应该导致程序停止运行。为了解决您的代码不应该干扰的那些异常和其他异常，except关键字可以接受一种或多种应该被明确抓住。其他任何错误都会被抛出，好像您根本没有try块一样。这将except块集中在那些绝对应处理的情况上，因此您的代码只需处理应处理的内容。对您刚刚尝试的内容进行小的更改，如下所示，以查看实际效果

```python
def count_lines(file_name: str) -> int:
    try:
        return len(open(file_name, "r").readlines())
    except IOError:
        print('exception error reading the file or calculating lines!')
        return 0


my_file = input("Enter file to open:")
print(count_lines(my_file))
```

By changing the code to accept IOError explicitly, the except block will only execute if there was a problem accessing the file from the filesystem. Any other errors, such as a filename that’s not even a string, will be raised outside of this function, to be handled by some other piece of code in the call stack.If you need to catch multiple exception types, there are two approaches. The first and easiest is to simply catch some base class that all the necessary exceptions derive from. Because exception handling matches against the specified class and all its subclasses, this approach works quite well when all the types you need to catch do have a common base class. In the line counting example, you could encounter either IOError or OSError, both of which are descendants of EnvironmentError

通过将代码更改为显式接受IOError，只有在从文件系统访问文件时遇到问题时，except块才会执行。其他任何错误（例如文件名甚至不是字符串）都将在此函数之外引发，并由调用堆栈中的其他一些代码处理。如果需要捕获多种异常类型，则有两种方法。第一个也是最简单的方法是简单地捕获一些所有必要异常都源自的基类。因为异常处理与指定的类及其所有子类匹配，所以当您需要捕获的所有类型都具有一个公共基类时，此方法将非常有效。在行计数示例中，您可能会遇到IOError或OSError，它们都是EnvironmentError的后代

```python
def count_lines(file_name: str) -> int:
    try:
        return len(open(file_name, "r").readlines())
    except EnvironmentError:
        print('exception error reading the file or calculating lines!')
        return 0


my_file = input("Enter file to open:")
print(count_lines(my_file))
```

除了捕获错误的基类以外，还可以获取多个错误类，比如仅捕获EnvironmentError和TypeError

```python
def count_lines(file_name: str) -> int:
    try:
        return len(open(file_name, "r").readlines())
    except (EnvironmentError,TypeError):
        print('exception error reading the file or calculating lines!')
        return 0


my_file = input("Enter file to open:")
print(count_lines(my_file))
```

在捕获的同时赋值变量
```python
import logging


def count_lines(file_name: str) -> int:
    try:
        return len(open(file_name, "r").readlines())
    except (EnvironmentError,TypeError) as e:
        print('exception error reading the file or calculating lines!')
        logging.error(e)
        return 0


my_file = input("Enter file to open:")
print(count_lines(my_file))
```

根据不同的错误类型做对应的处理
```python
import logging


def count_lines(file_name: str) -> int:
    try:
        return len(open(file_name, "r").readlines())
    except TypeError as e:
        logging.error(e)
        return 0
    except EnvironmentError as e:
        logging.error(e.args[1]) # ERROR:root:No such file or directory
        logging.error(e.args)  # ERROR:root:(2, 'No such file or directory')
        return 0


my_file = input("Enter file to open:")
print(count_lines(my_file))
```

### 异常链
Sometimes, while handling one exception, another exception might get raised along the way. This can happen either explicitly with a raise keyword or implicitly through some other code that gets executed as part of the handling. Either way, this situation brings up the question of which exception is important enough to present itself to the rest of the application. Exactly how that question is answered depends on how the code is laid out, so let’s take a look at a simple example, where the exception handling code opens and writes to a log file
If anything should go wrong when writing to the log, a separate exception will be raised. Even though this new exception is important, there was already an exception in play that shouldn’t be forgotten. To retain the original information, the file exception gains a new attribute, called __context__, which holds the original exception object. Each exception can possibly reference one other, forming a chain that represents everything that went wrong, in order. Consider what happens when get_value() fails, but logfile.txt is a read-only file:
有时，在处理一个异常时，可能会在过程中引发另一个异常。这可以通过使用raise关键字显式地发生，也可以通过作为处理的一部分而执行的其他代码隐式地发生。无论哪种方式，这种情况都会引发一个问题，即哪个异常足够重要，足以将其自身呈现给应用程序的其余部分。究竟该问题的答案取决于代码的布局方式，所以让我们看一个简单的示例，其中打开异常处理代码并将其写入日志文件
如果在写入日志时出现任何问题，将引发单独的异常。尽管这个新例外很重要，但在游戏中已经有一个不容忘记的例外。为了保留原始信息，文件异常将获得一个名为__context__的新属性，该属性保存原始异常对象。每个异常都可以相互引用，形成一条链，按顺序代表所有出错的事物。考虑一下get_value（）失败但logfile.txt是只读文件时会发生什么：

异常链的组成通常是因为多个异常组成，异常的报错流程是程序的执行流程
```python
def get_value(dictionary, name):
    try:
        return dictionary[name]
    except Exception as e:
        print("exception hit..writing to file")
        log = open('../temp/log.txt', 'w')
        log.write('%s\n' % e)
        log.close()


names = {"jack": 12, "rose": 13}
get_value({}, 'test')
```
修改`log.txt`的权限为066,报错内容如下
```bash 
exception hit..writing to file
Traceback (most recent call last):
  File "4-exceptionChains.py", line 13, in get_value
    return dictionary[name]
KeyError: 'test'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "4-exceptionChains.py", line 22, in <module>
    get_value({}, 'test')
  File "4-exceptionChains.py", line 16, in get_value
    log = open('../temp/log.txt', 'w')
PermissionError: [Errno 13] Permission denied: '../temp/log.txt'
```
这是一条隐式链，因为异常仅根据执行期间遇到异常的方式进行链接。有时您会自己生成一个异常，并且可能需要包括在其他地方生成的异常。一个常见的例子是使用传入的函数来验证值。如第3章和第4章所述，验证函数通常会引发ValueError，而不管出了什么问题。这是形成显式链的绝佳机会，因此我们可以直接引发ValueError，同时保留幕后的实际异常。 Python通过在raise语句的末尾包含from关键字来允许此操作：


> 根据异常链 捕获最近的异常

```python
def validate(value, validator):
    try:
        return validator(value)
    except Exception as e:
        raise ValueError('Invalid value: %s' % value) from e


def validator(value):
    if len(value) > 10:
        raise ValueError("Value can't exceed 10 characters")


if __name__ == '__main__':
    validate('test', validator)
    validate(False, validator)

```
报错内容如下
```bash
Traceback (most recent call last):
  File "/Users/alpaca/gitProject/ProPythonRecord/chapter2/4-exceptionChains.py", line 28, in validate
    return validator(value)
  File "/Users/alpaca/gitProject/ProPythonRecord/chapter2/4-exceptionChains.py", line 34, in validator
    if len(value) > 10:
TypeError: object of type 'bool' has no len()

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/Users/alpaca/gitProject/ProPythonRecord/chapter2/4-exceptionChains.py", line 40, in <module>
    validate(False, validator)
  File "/Users/alpaca/gitProject/ProPythonRecord/chapter2/4-exceptionChains.py", line 30, in validate
    raise ValueError('Invalid value: %s' % value) from e
ValueError: Invalid value: False
```

### 异常报错的完整处理
```python
import logging


def count(file_name: str) -> int:

    file = None
    try:
        file = open(file_name, 'r')
        lines = file.readlines()
    except EnvironmentError as e:
        logging.error(e)
        return 0
    except TypeError as e:
        logging.error(e)
        return 0
    except UnicodeDecodeError as e:
        "The contents of the file were in an unknown encoding"
        logging.error(e)
        return 0
    else:
        return len(lines)
    finally:
        if file:
            file.close()
```

### 条件表达式的优化
Fairly often, you may find yourself needing to access one of two values, and which one you use depends on evaluating an expression. For instance, it’s quite common to display one string to a user if the value exceeds a particular value and a different one otherwise. Typically, this would be done using an if/else combination, as here:
Rather than writing this out into four separate lines, it’s possible to condense it into a single line using a conditional expression. By converting the if and else blocks into clauses in an expression, Python does the same effect much more concisely
```python
def test_value(value):
    if value < 100:
        return 'the value is just right'
    else:
        return 'the value is too big'


test_value(20)
```
通常，您可能会发现自己需要访问两个值之一，而使用哪个值取决于对表达式的求值。例如，如果值超过特定值，则向用户显示一个字符串，否则显示不同的字符串，这很常见。通常，这将使用if / else组合完成，如下所示
除了将其写成四行外，还可以使用条件表达式将其压缩为一行。通过将if和else块转换为表达式中的子句，Python可以更简洁地实现相同的效果

```python
def test_value(value: int):
    return 'the value' + ('is just right' if value < 100 else 'is too  big')
```

if you’re used to this behavior from other programming languages, python’s ordering may seem unusual at first. Other languages, such as C++, implement something of the form expression ? value_1 : value_2.  that is, the expression to test comes first, followed by the value to use if the expression is true, then the value to use if the expression is false.instead, python attempts to use a form that more explicitly describes what’s really going on. the expectation is that the expression will be true most of the time, so the associated value comes first, followed by the expression, then the value to use if the expression is false. this takes the entire statement into account by putting the more common value in the place it would be if there were no expression at all. For example, you end up with things like return value ... and x = value ....Because the expression is then tacked on afterward, it highlights the notion that the expression is just a qualification of the first value. “Use this value whenever this expression is true; otherwise, use the other one.” it may seem a little odd if you’re used to another language, but it makes sense when thinking about the equivalent in plain english
如果您已经习惯了其他编程语言的这种行为，那么起初python的排序可能看起来并不常见。其他语言（例如C ++）实现某种形式的表达式？ value_1：value_2。也就是说，首先要测试的表达式，然后是表达式为true时要使用的值，然后是表达式为false时要使用的值。相反，python尝试使用更明确地描述实际情况的形式。期望表达式在大多数情况下都是true，因此关联的值首先出现，然后是表达式，如果表达式为false，则是要使用的值。通过将更通用的值放在根本没有表达式的位置，从而将整个语句考虑在内。例如，您最终会遇到诸如return value ...和x = value ...之类的事情。由于随后加上了表达式，因此突出了以下概念：表达式只是第一个值的限定条件。 “只要此表达式为真，就使用此值；否则，请使用另一个。”如果您习惯另一种语言，这似乎有些奇怪，但是在考虑普通英语中的等效语言时，这是有道理的

还有另外一种使用and和or来做逻辑判断的
```python
def test_value(value: int):
    return 'the value' + ( value < 100 and ' is just right' or 'is too big')

print(test_value(20))
```


### iteration
range返回的是一个迭代的对象，每次返回一个object,使用`help(range)`的时候可以看下，但是可以使用list(range(0,5))返回一个列表\

There are generally two ways of looking at sequences: as a collection of items, or as a way to access a single item at a time. These two aren’t mutually exclusive, but it’s useful to separate them in order to understand the different features available in each case. Working on the collection as a whole requires that all the items be in memory at once, but accessing them one at a time can often be done much more efficiently.Iteration refers to this more efficient form of traversing a collection, working with just one item at a time before moving on to the next. Iteration is an option for any type of sequence, but the real advantage comes in special types of objects that don’t need to load everything in memory all at once. The canonical example of this is Python’s built-in range() function, which appears to iterate over the integers that fall within a given range
通常，有两种查看序列的方式：作为项目的集合，或作为一次访问单个项目的方式。两者并非互斥，但将它们分开以了解每种情况下可用的功能很有用。整体上处理集合需要将所有项目同时存储在内存中，但一次访问一个项目通常可以更高效地进行。迭代是一种遍历集合的高效形式，仅处理一个项目一次移至下一个。迭代是任何类型的序列的一个选项，但是真正的优势在于特殊类型的对象，它们不需要一次将所有内容都加载到内存中。一个典型的例子是Python的内置range（）函数，该函数似乎在给定范围内的整数上进行迭代
At a glance, it may appear like range() returns a list containing the appropriate values, but it doesn’t. This shows if you examine its return value on its own, without iterating over it
乍一看，它看起来像range（）返回包含适当值的列表，但事实并非如此。这显示了您是否自行检查其返回值，而无需对其进行迭代
The range object itself doesn’t contain any of the values in the sequence. Instead, it generates them one at a time, on demand, during iteration. If you truly want a list that you can add or remove items from, you can convert one by passing the range object into a new list object. This internally iterates just like a for loop, so the generated list uses the same values that are available when iterating over the range itself.Chapter 5 shows how you can write your own iterable objects that work similarly to range(). In addition to providing iterable objects, there are a number of ways to iterate over these objects in different situations, for different purposes. The for loop is the most obvious technique, but Python offers other forms of syntax as well, which are outlined in this section
range本身不包含序列中的任何值。相反，它在迭代过程中按需一次生成一个。如果您确实想要一个可以添加或删除项目的列表，则可以通过将范围对象传递到新的列表对象来进行转换。这在内部像for循环一样进行迭代，因此生成的列表使用在范围本身上进行迭代时可用的相同值。第5章说明了如何编写自己的可迭代对象，这些对象的工作方式与range（）类似。除了提供可迭代的对象外，还有多种方法可以在不同情况下出于不同目的在这些对象上进行迭代。 for循环是最明显的技术，但是Python也提供了其他形式的语法，本节对此进行了概述。

```python
for i in range(9, 100):
    print(i)


print(list(range(9, 100)))
```

### 序列解包
通常对于少量的 固定的返回的变量内容时候，可以在=号左边直接赋值，通常一个函数返回多个变量的时候也可以这样操作,当遇到不确定的返回数量的时候可以使用`*变量来代替`
```python
url = "example.com"
prefix, suffix = url.split(".")
print(prefix, suffix)


def return_data():
    return 'hzj', 12


name, age = return_data()


def return_datas():
    return 'hzj', 12, 23, 45, 56


name, *ages = return_datas()
print(name, type(ages))  # hzj <class 'list'>

```

### List  Comprehensions
简化代码
```python
output = []

for value in range(10, 100):
    if value < 20:
        output.append(str(value))


# 简写
output = [str(value) for value in range(10, 100) if value < 20]
print(min([value for value in range(1, 100) if value < 20]))
```


### Generator  Expressions
Instead of creating an entire list based on certain criteria, it’s often more useful to leverage the power of iteration for this process as well. Instead of surrounding the compression in brackets, which would indicate the creation of a proper list, you can instead surround it in parentheses, which will create a generator. Here’s how it looks in action:
与其根据某些标准创建整个列表，不如利用迭代的力量来完成此过程通常更为有用。可以将压缩内容放在括号中，这将创建一个生成器，而不必将压缩内容括在方括号中，以表明已创建了正确的列表。这是实际效果

```python
gen = (value for value in range(10, 30) if value > 20)
print(type(gen)) # <class 'generator'>


dict = {value: str(value) for value in range(10) if value > 5}
print(dict)  # {6: '6', 7: '7', 8: '8', 9: '9'}


import itertools

print(list(itertools.chain(range(3), range(4)))) # [0, 1, 2, 0, 1, 2, 3]


print(list(zip(range(3), reversed(range(10)))))  # [(0, 9), (1, 8), (2, 7)]

```


### collection
```python
from collections import namedtuple

Point = namedtuple("Point", 'x y')
point = Point(13, 25)
print(point) # Point(x=13, y=25)
print(point.x)
print(point.y)
```

如果想让字典有序，可以使用collections.OrderedDict，它现在在C中实现，这使其快4到100倍。
```python
from collections import OrderedDict

od=OrderedDict()
print(isinstance(od,OrderedDict)) # True
print(isinstance(od,dict)) # True

from collections import OrderedDict
d = OrderedDict((value, str(value))for value in range(10) if value > 1)
print(d)  # OrderedDict([(2, '2'), (3, '3'), (4, '4'), (5, '5'), (6, '6'), (7, '7'), (8, '8'), (9, '9')])

```

如果获取不到字典中的值，则变量保存默认值 使用get
```python
dict = {"name": "hzj", "age": 12}
print(dict.get("name"))  # hzj
print(dict.get("address", "hangzhou"))  # hangzhou


def count_words(text):
    count = {}
    for word in text.split(' '):
        current = count.get(word, 0)
        count[word] = current + 1
    return count


print(count_words("a b bcb bdsb adabdbcvbs dsb ca a"))
# {'a': 2, 'b': 1, 'bcb': 1, 'bdsb': 1, 'adabdbcvbs': 1, 'dsb': 1, 'ca': 1}


from collections import defaultdict


def count_words2(text):
    count = defaultdict(int)
    for word in text.split(' '):
        count[word] += 1
    return count


print(count_words2("a b bcb bdsb adabdbcvbs dsb ca a"))
# defaultdict(<class 'int'>, {'a': 2, 'b': 1, 'bcb': 1, 'bdsb': 1, 'adabdbcvbs': 1, 'dsb': 1, 'ca': 1})
```


### 兼容不同版本的包引用
```python
try:
    from hashlib import md5
except ImportError
    from md5 import new as md5
```

### 使用__all__自定义导入包
比如现在有一个`xx.py`的文件,`a.py`想要引用`xx.py`文件中的内容
```python
# xx.py

def func1():
    return 1000

def func2():
    return 20000

# a.py
from chapter2.xx import *

if __name__ == '__main__':
    print(func1()) # 1000
    print(func2()) # 20000
```
这样是可以完全引用的，但有时候我们需要导入部分包，这个时候可以用`__all__`来管理，如下
```python
# xx.py
__all__ = ["func1"]
def func1():
    return 1000

def func2():
    return 20000

# a.py
from chapter2.xx import *

if __name__ == '__main__':
    print(func1()) # 1000
    print(func2()) # error  NameError: name 'func2' is not defined
```

### The __import__() function
You don’t always have to place your imports at the top of a module. In fact, sometimes you might not be able to write some of your imports in advance at all. You might be making decisions about which module to import based on user-supplied settings, or perhaps you’re even allowing users to specify modules directly. These user-supplied settings are a convenient way to allow for extensibility without resorting to automatic discovery.Chapter 2  advanCed BasiCs
In order to support this functionality, Python allows you to import code manually using the `__import__()` function. It’s a built-in function, so it’s available everywhere, but using it requires some explanation because it’s not as straightforward as some of the other features provided by Python. You can choose from five arguments to customize how a module gets imported and what contents are retrieved
- name: The only argument that is always required, this accepts the name of the module that should be loaded. If it’s part of a package, just separate each part of the path with a period, as when using import path.to.module
- globals: A namespace dictionary that is used to define the context in which the module name is resolved. In standard import cases, the return value from the built-in globals() function is used to populate this argument
- locals: Another namespace dictionary, ideally used to help define the context in which the module name is resolved. In reality, however, current implementations of Python simply ignore it. In the event of future support, the standard import provides the return value from the built-in locals() function for this argument
- fromlist: A list of individual names that should be imported from the module, rather than importing the full module.
- level: An integer indicating how the path should be resolved with respect to the module that calls `__import__()`. A value of -1 allows both absolute and implicit relative imports; 0 allows only absolute imports; positive values indicate how many levels up the path to use for an explicit relative import
Even though that may seem simple enough, the return value contains a few traps that can cause quite a bit of confusion. It always returns a module object, but it can be surprising to see which module is returned and what attributes are available on it. Because there are a number of different ways to import modules, these variations are worth understanding. First, let’s examine how different types of module names impact the return value.
In the simplest case, you’d pass in a single module name to `__import__()`, and the return value is just what you’d expect: the module referenced by the name provided. The attributes available on that module object are the same as you’d have available if you imported that name directly in your code: the entire namespace that was declared in that module’s code
When you pass in a more complex module path, however, the return value may not match expectations. Complex paths are provided using the same dot-separated syntax used in your source files, so importing os.path, for instance, would be achieved by passing in "os.path". The returned value in that case is os, but the path attribute lets you access the module you’re really looking for.
The reason for that variation is that `__import__()` mimics the behavior of Python source files, in which import os.path makes the os module available under that name. You can still access os.path, but the module that goes into the main namespace is os. Because `__import__()` works essentially the same way as a standard import, what you get in the return value is what you would have in the main module namespace ordinarily
In order to get just the module at the end of the module path, you can take a couple of different approaches. The most obvious, although not necessarily direct, would be to split the given module name on periods, using each portion of the path to get each attribute layer from the module returned by `__import__()`. Here’s a simple function that would do the job
您不必总是将导入内容放在模块的顶部。实际上，有时您可能根本无法事先编写一些导入文件。您可能根据用户提供的设置来决定要导入哪个模块，或者甚至允许用户直接指定模块。这些用户提供的设置是一种无需扩展即可自动进行扩展的便捷方法。第2章高级BasiC




### Python导包
因为idea设置的问题，可能会导致可以引用包，但是无法运行的问题，运行的时候回报错
```bash
🏃:chapter2/ (master✗) $ python 11-Import.py 
Traceback (most recent call last):
  File "11-Import.py", line 17, in <module>
    from chapter2.xx import *
ModuleNotFoundError: No module named 'chapter2'
```

解决办法1 直接在程序中用`if __name__ == __main__:` 运行
解决办法2 设置idea 设置内容见后
https://www.cnblogs.com/qi-yuan-008/p/12827918.html
https://zhuanlan.zhihu.com/p/110407087