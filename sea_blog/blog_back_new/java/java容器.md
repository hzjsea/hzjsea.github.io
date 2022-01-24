---
title: java容器
categories:
  - java
tags:
  - java
abbrlink: 17ea147a
date: r2020-10-27 11:20:16
---

![2020-10-27-11-20-57](http://noback.upyun.com/2020-10-27-11-20-57.png)
<!-- more -->

# java容器
java中的容器主要是一对存储对象，不同类型的存储对象具有不同的存储特点，经过不断地优化，已经十分的完善了
从图中可以看出，java的容器分为`Collection`和`Map`两种类型，他们两个的区别在于，存储单组对象的时候，对象的数量不同。
- collection 主要是单个元素的集合，由List、Queque、Set 三个接口区分特征
- Map 存储一组键值对，存储对象为2个

## List
list的特点是所有的元素是可以`重复`的,List接口在Collection的基础上增加了很多的方法
List主要分为ArrayList和LinkedList，前者底层是使用`数组`实现的List，后者是使用`链表`实现的List。
一种是顺序存储，另一种则是链式存储。

Vector是一个已经被弃用的类，因为他是线程同步的，而我们平时使用的时候都是非同步的，使用同步的坏处就是会在一个记录上加锁，防止多个程序访问同一条数据导致数据不同步。这样会导致访问速度变慢。

Stack是满足“后进先出”规则的容器，注意LinkedList可以实现所有的栈功能。


###  ArrayList
ArrayList是一个可以动态增长的数组,java中的数组一旦指定了长度就不可以改变，在业务中大都使用的是arraryList 数组
ArrayList默认的长度是10，如果我们插入的数据超过了10，ArrayList会不断的自我增长。
ArrayList由于底层是使用数组实现的，所以随机访问速度快(O1)，插入删除较慢(0n)

### Array
Array是一个静态数组，在申明数组的时候就要初始化并确定长度，而且他只能存储同一类型的数据
将LinkList转换为Arrary toArrary

### LinkList
LinkedList是使用链表实现的容器。链式存储
`在列表中插入和删除速度快，但是查找需要遍历整个链表，速度较慢`
使用LinkedList可以实现很多队列、栈的数据结构，并且有很多方法很类似，但是有细小的差别
主要使用的方法如下
```java
getFirst,element
peek
remove,removeFirst
poll
```

```java
public class mycollection1 {

    public static void main(String[] args) {
        LinkedList<String> lc1 = new LinkedList<>();
        // 添加元素
        lc1.add("hel");
        // 指定位置添加元素
        lc1.add(1,"hello");
        lc1.add(1,"word");
        System.out.println(lc1.toString());  //  [hel, word, hello]
        // 获取列表头元素,如果列表为空，则抛出异常
        System.out.println(lc1.element());  //   hel
        // 获取列表头元素，如果列表为空，则抛出异常
        System.out.println(lc1.getFirst()); //  hel
        // 获取列表头元素，如果列表为空，则返回null
        System.out.println(lc1.peek());
        // 删除列表头元素，并返回，如果列表为空，则抛出异常
        System.out.println(lc1.remove()); // hel
        System.out.println(lc1.removeFirst());  // word
        // poll 删除并返回，如果为空，则返回null
        System.out.println(lc1.poll()); // hello
    }

}
```

使用LinkedList可以实现栈
```java
public class MyStack<T> {

    private LinkedList<T> storage = new LinkedList<T>();

    /*
     * 进栈
     */
    public void push(T v){
        storage.addFirst(v);
    }

    /**
     * 窥视栈顶
     * @return
     */
    public T peek(){
        return storage.getFirst();
    }

    /**
     * 出栈
     * @return
     */
    public T pop(){
        return storage.removeFirst();
    }

    public boolean empty(){
        return storage.isEmpty();
    }

    public String toString(){
        return storage.toString();
    }
}  
```