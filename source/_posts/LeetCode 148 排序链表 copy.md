---
title: LeetCode 148 排序链表
date: 2024-09-02 16:41:14
tags: [LeetCode,算法,链表,刷题]
---

## 题目
https://leetcode.cn/problems/sort-list/description/

给你链表的头结点 `head` ，请将其按 `升序` 排列并返回 `排序后的链表` 。

![](https://pic.imgdb.cn/item/66d59265d9c307b7e9f41e86.png)

## 思路
- 归并排序
- 二分法
- getMiddle() 获取中间节点
- merge(ListNode left, ListNode right)  合并两个有序链表

考点归并排序，只不过排序的对象换成了ListNode。
归并就是二分法，递归树的叶子节点变成了单个元素，在回溯的时候用merge函数去比较大小。

可以拆解为3个问题
1、找出中间节点，拆分成left和right
2、然后排序left、排序right
3、然后合并left和right。

实现了gitMiddle函数和merge函数，问题就解决了。