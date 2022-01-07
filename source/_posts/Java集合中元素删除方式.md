---
title: Java集合中元素删除方式
date: 2021-06-29 12:03:45
tags:
- java
- 集合
categories:
- Java
---

# 简介

删除集合中的元素，有两种删除的形式，一种是删除特定元素，一种是删除特定索引的元素。

删除的方式有：使用Java API （java 8）、从后往前的循环、使用迭代器、使用新的集合。

<!-- more -->

# Java API

- `List#remove(int index)`。最基本API，删除特定索引处元素，O(1)。
- `List#remove(Object o)`。从前往后遍历，移除第一个与 `o` 相等的元素，O(n)。
- `Collection#removeIf(Predicate<? super E> filter)`。移除所有特定元素，内部使用迭代器；Java 8；O(n)。
- `ArrayList#removeIf(Predicate<? super E> filter)`。移除所有特定元素，内部使用`BitSet`标记待删除元素；Java 8；O(n)。
- `List#subList(startIndex, endIndex).clear()`。删除特定索引区间的元素，O(n)。

>`BitSet`利用`long`数组来维护自身，数组中的每个长整型代表64个位元素，每个元素的值为`True`或`False`。常用于标记其他集合。

# 从后往前循环

适用两种形式：删除特定元素，以及删除特定索引的元素。

```java
// 删除所有特定元素
for (int index = list.size() - 1; index >= 0; index --) {
    Object element = list.get(index);
    if (special.equals(element)) {
        list.remove(index);
    }
}

// 删除多个特定索引的元素 比如，删除索引在[start, end]内的元素
for (int index = end; index >= start; index --) {
    list.remove(index);
}
```

# 使用迭代器

适用删除特定元素；部分（元素可标记）适用删除特定索引元素，先标记再迭代删除。

```java
// 删除所有特定元素
Iterator<Integer> iterator = list.listIterator();
while (iterator.hasNext()) {
    Integer element = iterator.next();
    if (special.equals(element)) {
        iterator.remove();
    }
}

// 删除特定索引元素，元素可标记，比如元素里的ID不能为负数，标记为负数即为待删除元素
// 比如删除索引在[start, end]中的整数，如果集合不为负数，则可以标记待删除元素为负数。
for (int index = start; index <= end; index ++) {
    list.set(index, Integer.MIN_VALUE);
}
Iterator<Integer> iterator = list.listIterator();
while (iterator.hasNext()) {
    Integer element = iterator.next();
    if (element == Integer.MIN_VALUE) {
        iterator.remove();
    }
}
```

# 使用新的集合

创建新的集合，循环将旧集合中有效元素添加到新集合中。在实践中也较常用。

# 总结

实践中，各种方法的复杂度都相同，使用新的集合需要额外的空间，如果集合比较大则效率较低。

优先考虑使用API，不能满足的情况下再考虑向前循环、迭代或创建新的集合。

