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

- [删除排序链表中的重复元素](#删除排序链表中的重复元素)
  - [问题](#问题)
  - [C语言实现](#c语言实现)
  - [python 语言实现](#python-语言实现)

<!-- /code_chunk_output -->
<!-- more -->


# 删除排序链表中的重复元素


## 问题
```c
// 给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。

// 返回删除后的链表的头节点。

// example1:
// 输入: head = [4,5,1,9], val = 5
// 输出: [4,1,9]
// 解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9

// example2:
// 输入: head = [4,5,1,9], val = 1
// 输出: [4,5,9]
// 解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9
```

## C语言实现

```c
// 2020-07-21 23:53:11
#include <stdio.h>
#include <stdlib.h>

struct ListNode
{
    int val;
    struct ListNode *next;
};

struct ListNode* deleteNode(struct ListNode* head, int val){
    if(head == NULL) {
        return NULL;
    }
    if(head->val == val) {
        return head->next;
    } 
    
    // 创建一个游标
    struct ListNode *current = head;
    while((current->next!=NULL && current->next->val !=val)){
        current = current->next;
    }
    if (current->next !=NULL){
        current -> next = current->next->next;
    }
    return head;

}
```

## python 语言实现

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

class Solution:
    def deleteNode(self, head: ListNode, val: int):
        cur = head
        if head is None:
            return 0
        if head.val == val:
            return head.next
    
        while (cur.next is not None and cur.next.val != val):
            cur = cur.next
        
        # 判断是不是最后一位，如果cur是最后一位的话 就没有next了，也就是不存在重复值、
        if cur.next is not None:
            cur.next = cur.next.next
        # cur.next = cur.next.next # 但这样也能通过..
        return head
```

