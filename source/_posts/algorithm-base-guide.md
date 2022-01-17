---
title: 数据结构和算法-概览
date: 2020-01-16 15:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 简介

**数据结构**

数据结构的存储方式只有两种：数组（顺序存储）和链表（链式存储），其他的都可以用这两者来实现，比如队列、栈、图、散列表、树等。

数据结构的遍历和访问：线性方式即迭代，非线性方式即递归（或队列栈加迭代）。

**算法**

核心在于穷举，然后剪枝。剪枝可利用记录表。

<!--more-->

# 简单示例-翻转链表

给定单链表的头节点 `head` ，请反转链表，并返回反转后的链表的头节点。

**迭代**

```java
public ListNode reverseList(ListNode head) {
    ListNode cur = head;
    ListNode pre = null;
    while(cur != null) {
        ListNode next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return pre;
}
```



**递归**

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode newHead = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

# 参考

- [数据结构](https://zh.wikipedia.org/wiki/数据结构)
- [数据结构术语列表](https://zh.wikipedia.org/wiki/数据结构术语列表)
- [如何系统地学习数据结构与算法？](https://zhuanlan.zhihu.com/p/137041568)
- [LeetCode刷题手册](https://books.halfrost.com/leetcode/)：比较系统的刷题
- https://leetcode-cn.com/problems/UHnkqh/
