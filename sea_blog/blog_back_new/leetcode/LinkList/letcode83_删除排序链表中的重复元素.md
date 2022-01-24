---
title: leetcode_删除排序链表中的重复元素
categories:
  - leetcode
tags:
  - leetcode
abbrlink: c2cc922b
date: 2020-07-21 09:57:46
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
```bash
# 给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。
# example1
输入: 1->1->2
输出: 1->2

# example2
输入: 1->1->2->3->3
输出: 1->2->3

#example3
输入: 1->1->1
输出: 1
```

## C语言实现

```c
struct ListNode *deleteDuplicates(struct ListNode *head)
{
    struct ListNode *current = head;
    while (current != NULL && current->next !=NULL)
    // while (current->next != NULL && current != NULL)
    {
        // 这样写会内存溢出呀
        if (current->val == current->next->val)
        {
            current->next = current->next->next;
        }else
        {
            current = current->next;
        }
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
    def deleteDuplicates(self, head: ListNode) -> ListNode:
        current =head
        while current and current.next:
            if current.val == current.next.val:
                current.next = current.next.next
            else:
                current = current.next
        return head
```

