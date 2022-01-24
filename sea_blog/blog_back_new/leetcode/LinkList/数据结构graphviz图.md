---
title: 数据结构graphviz图片
categories:
  - leetcode
tags:
  - leetcode
abbrlink: aeae5308
date: 2020-07-19 18:17:06
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [数据结构](#数据结构)
  - [链表](#链表)
    - [单链表](#单链表)
    - [单链表增删改查](#单链表增删改查)
    - [双向链表结构图](#双向链表结构图)
    - [双向链表的增删改查](#双向链表的增删改查)
  - [顺序表](#顺序表)
    - [反转链表](#反转链表)

<!-- /code_chunk_output -->
<!-- more -->

# 数据结构

## 链表

### 单链表

```dot
digraph G {
    label="单链表结构"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{data|next}"];
    node2 [label="{data|next}"];
    node3 [label="{data|next}"];
    node4 [label="{data|next}"];

    node1 -> node2;
    node2 -> node3;
    node3 -> node4;
    node4 -> null;


}
```

### 单链表增删改查

```dot
digraph G {
    label="单链表添加节点"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{data|next}"];
    node2 [label="{data|next}"];
    node3 [label="{data|next}"];
    node4 [label="{data|next}"];

    node1 -> node2;
    node2 -> node3 [label="③" style=dashed color="red"];
    node3 -> node4;
    node4 -> null

    // 增加过程
    newnode [label="{data|next}"];
    newnode:next -> node3 [label="①" color="red"];
    node2:next -> newnode:next [label="②" color="red"];

}
```

```dot
digraph G {
    label="单链表删除节点"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{data|next}"];
    node2 [label="{data|next}"];
    node3 [label="{data|next}" style="dashed" color="red"];
    node4 [label="{data|next}"];

    node1 -> node2;
    node2 -> node3 [label="②" color="red" style="dashed"]
    node3 -> node4 [label="③" color="red" style="dashed"]
    node4 -> null

    node2:next -> node4 [label="①",color="red"];
}
```

```dot
digraph G {
    label="单链表修改节点"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{data|next}"];
    node2 [label="{data|next}"];
    node3 [label="{data|next}"];
    node4 [label="{data|next}"];

    node1 -> node2;
    node2 -> node3
    node3 -> node4
    node4 -> null

    // 修改过程
    node3:data -> node3:data [label="" color="red"]



}
```

```dot
digraph G {
    label="单链表查找节点"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{data|next}"];
    node2 [label="{data|next}"];
    node3 [label="{data|next}"];
    node4 [label="{data|next}"];

    node1 -> node2;
    node2 -> node3
    node3 -> node4
    node4 -> null


    current -> node1 [label="1" color="red" style="dashed"]
    current -> node2 [label="2" color="red" style="dashed"]
    current -> node3 [label="3" color="red" style="dashed"]
    current -> node4 [label="4" color="red" ]
}
```

### 双向链表结构图

```dot
digraph G {
    label="单链表结构"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{prev|data|next}"];
    node2 [label="{prev|data|next}"];
    node3 [label="{prev|data|next}"];
    node4 [label="{prev|data|next}"];

    node1:next -> node2:data;
    node2:next -> node3:data;
    node3:next -> node4:data;
    node4:next -> null;

    node4:prev -> node3:data;
    node3:prev -> node2:data;
    node2:prev -> node1:data;

}
```

### 双向链表的增删改查

```dot
digraph G {
    label="双链表增加"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{prev|data|next}"];
    node2 [label="{prev|data|next}"];
    node3 [label="{prev|data|next}"];
    node4 [label="{prev|data|next}"];

    node1:next -> node2:data ;
    node2:next -> node3:data [label="5" color="red" style="dashed"];
    node3:next -> node4:data;
    node4:next -> null;

    node4:prev -> node3:data;
    node3:prev -> node2:data [label="6" color="red" style="dashed"]
    node2:prev -> node1:data;

    // lable="增加过程"
    newnode [label="{prev|data|next}|newnode"];
    newnode:prev -> node2:data [label="1" color="red"]
    newnode:next -> node3:data [label="3" color="red"]
    node2:next -> newnode:data [label="2" color="red"]
    node3:prev -> newnode:data [label="4" color="red"]

}
```

```dot
digraph G {
    label="双链表删除"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{prev|data|next}"];
    node2 [label="{prev|data|next}"];
    node3 [label="{prev|data|next}"];
    node4 [label="{prev|data|next}"];

    node1:next -> node2:data [label="3" color="red" style="dashed"]
    node2:next -> node3:data [label="4" color="red" style="dashed"]
    node3:next -> node4:data;
    node4:next -> null;

    node4:prev -> node3:data;
    node3:prev -> node2:data [label="5" color="red" style="dashed"]
    node2:prev -> node1:data [label="6" color="red" style="dashed"]

    // 删除过程
    node1:next -> node3:data [label="1" color="red" ]
    node3:prev -> node1:data [label="2" color="red" ]


}
```

```dot
digraph G {
    label="双链表改"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{prev|data|next}"];
    node2 [label="{prev|data|next}"];
    node3 [label="{prev|data|next}"];
    node4 [label="{prev|data|next}"];

    node1:next -> node2:data
    node2:next -> node3:data
    node3:next -> node4:data;
    node4:next -> null;

    node4:prev -> node3:data;
    node3:prev -> node2:data
    node2:prev -> node1:data


    // 修改过程
    node3:data->node3:data [label="1" color="red" style="dashed"]

}
```

## 顺序表

```dot
digraph Gp{
    node [shape = record];
    node1 [label="node1|node2|node3"];
}
```



### 反转链表

```dot
digraph G {
    label="反转链表"
    rankdir=LR;
    node [shape = record];
    head -> node1;
    node1 [label="{data|next}"];
    node2 [label="{data|next}"];
    node3 [label="{data|next}"];
    node4 [label="{data|next}"];


    node3 -> node4;
    node4 -> null;

    prevnode [label="{data|next}"];

    head -> prevnode
    prevnode -> node2 [label="1" color="red"]
    node1 -> node3 [label="2" color="red"]
    node2 -> node1 [label="3" color="red"]

}
```
