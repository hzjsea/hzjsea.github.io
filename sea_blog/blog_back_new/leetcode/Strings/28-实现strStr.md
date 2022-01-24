---
title: 实现strStr
categories:
  - leetcode
tags:
  - leetcode
abbrlink: 3469c295
date: 2020-09-10 15:16:05
---


<!-- more -->
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
# 实现strStr

## 题目要求
给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)

## 解法1 思路
直接使用语言内置的方法
```go
package main

import (
	"strings"
)

func strStr(haystack string, needle string) int {
	return strings.Index(haystack,needle)
}
```