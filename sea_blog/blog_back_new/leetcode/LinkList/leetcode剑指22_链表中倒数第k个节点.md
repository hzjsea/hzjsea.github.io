---
title: leetcode22_链表中倒数第k个节点
categories:
  - leetcode
tags:
  - leetcode
abbrlink: 2b859953
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
// 解题要求
// 链表中倒数第k个节点
// 给定一个链表: 1->2->3->4->5, 和 k = 2.
// 返回链表 4->5.


// 解题思路
// 使用快慢指针，得出K的差距
// 当快指针的值=NULL的时候，慢指针所在节点就是倒数K的结点
// 需要返回的是慢指针所在节点的地址
```

## C语言实现

```c
#include <stdio.h>
#include <stdlib.h>
struct ListNode* getKthFromEnd(struct ListNode* head, int k){
    if(!head || k<=0){
        return NULL;
    }

    // 创建两个节点
    struct ListNode *pre = head;
    struct ListNode *last = head;

    // 创建快慢指针的差距
    while(k){
        last = last->next;
        // 判断是否只有一个节点
        if (!last && k !=1){
            return NULL;
        }
        k--;
    }
    // 让快慢指针不断向后跳，直到快指针=NUll
    while(last!=0){
        pre = pre->next;
        last = last->next;
    }
    return pre;
}
```

## python 语言实现

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None
        
class Solution:
    def getKthFromEnd(self, head: ListNode, k: int):
        # 使用快慢指针举例来解决问题
        fast = head
        slow = head
        while k:
            fast = fast.next
            k -=1

        # 此时fast与slow相距k距离，两点向后平移
        while fast:
            fast = fast.next
            slow = slow.next
        return slow
```

