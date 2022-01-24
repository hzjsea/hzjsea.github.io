---
title: leetcode剑指18_删除链表中的节点
categories:
  - leetcode
tags:
  - leetcode
abbrlink: 4c2add17
date: 2020-07-26 09:57:46
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [leetcode剑指18_删除链表中的节点](#leetcode剑指18_删除链表中的节点)
  - [问题](#问题)
  - [C语言实现](#c语言实现)
  - [python 语言实现](#python-语言实现)

<!-- /code_chunk_output -->
<!-- more -->


# leetcode剑指18_删除链表中的节点

## 问题
```c
// 定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

// example
// 输入: 1->2->3->4->5->NULL
// 输出: 5->4->3->2->1->NULL
```

## C语言实现

```c

// 2020-07-21 23:53:11

// 要求
// 反转一个单链表。
// 示例:
// 输入: 1->2->3->4->5->NULL
// 输出: 5->4->3->2->1->NULL
// 进阶:
// 你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

// 解题思路 
// 使用三指针 + 哨兵节点来解决
// 哨兵节点主要是在头结点前面放置一个节点
// 三指针分别表示  当前节点current 上一节点 pre  下一临时节点tmp(过渡使用)
#include <stdio.h>
#include <stdlib.h>
struct ListNode
{
    int val;
    struct ListNode *next;
};

struct ListNode* reverseList(struct ListNode* head){

    // 创建一个哨兵节点 指向null
    struct ListNode *pre = NULL;
    // 创建一个游标节点 初始位置为head
    struct ListNode *current = head;

    // 创建一个tmp节点
    struct ListNode *tmp = NULL;

    // 判断，如果链表为空链表或者只有一个节点，则返回头结点
    if (!head || head->next == NULL){
        return head;
    }

    // 利用迭代来解决问题
    // 迭代停止的终止条件为current->next = NULL or 0
    while(current){
        tmp =   current->next;
        current -> next  = pre;
        pre = current;
        current = tmp;
    }
    return head;
}




```

## python 语言实现

```python
class Solution:
    def reverseList(self, head: ListNode):
        if head is None or head.next is None :
            return head
        cur = head
        pre = None

        while cur:
            tmp = cur.next
            cur.next = pre
            pre = cur
            cur = tmp
        return pre
```

