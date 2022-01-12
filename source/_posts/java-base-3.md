---
title: Java基础三：集合
date: 2018-09-13 15:14:22
updated: 2022-01-11 11:52:48
tags:
- java
categories:
- Java
---

# `ArrayList`

基于数组，数组的增删操作使用方法`System.arraycopy()`，扩容机制为每次扩容1.5倍。

<!--more-->

```java
/**
  * ArrayList扩容的核心方法。
  */
private void grow(int minCapacity) {
    // oldCapacity为旧容量，newCapacity为新容量
    int oldCapacity = elementData.length;
    //将oldCapacity 右移一位，其效果相当于oldCapacity /2，结果为扩容1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //再检查新容量是否超出了ArrayList所定义的最大容量，
    //若超出了，则调用hugeCapacity()来比较minCapacity和 MAX_ARRAY_SIZE，
    //如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE。
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

# `LinkedList`

`LinkedList`是一个实现了List接口和Deque接口的双端链表，底层的链表结构使它支持高效的插入和删除操作，另外它实现了Deque接口，使得LinkedList类也具有队列的特性。

`LinkedList`基于内部的双端链表节点Node来实现。

# `HashMap` 

基于数组和链表，使用拉链法解决哈希冲突，链表长度大于阈值（默认为 8）时，链表转化为红黑树。

添加元素机制：HashMap 通过 key 的 hashCode 经过`hash()`方法处理过后得到 hash  值，然后通过 `(n - 1) & hash` 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

`hash()`方法：哈希值的高位与低位进行异或处理，同时具有高低位的特征，降低哈希冲突的概率。

```java
// jdk 1.8+
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^ ：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

## 扩容机制

扩容时机：在`put()`时判断是否进行扩容。`put()`方法开始时，如果数组为空则进行初始化扩容；`put()`方法末尾，判断元素个数是否超过阈值，超过就调用`resize()`方法进行扩容。

> `resize()`方法会遍历hash表中所有的元素，是非常耗时的，在编写程序中，要尽量避免`resize()`。

扩容过程：如果数组为空，则生成一个默认大小为16的Node数组；如果数组不为空，则**生成一个大小为旧数组大小两倍的新数组**，再将就数组元素逐个添加到新数组中，添加的方法为遍历数组，节点数据存在则判断

- 如果没有后继，则直接添加到hash & (newCap - 1)索引处；

- 如果有后继，则可能会把链表分成两部分，**低索引区和高索引区链表**，低索引区就是原来的索引位置，高索引区就是当前索引+oldCap的位置。划分的依据是`hash & oldCap == 0`，为true则放在低索引区，为false则放在高索引区。

