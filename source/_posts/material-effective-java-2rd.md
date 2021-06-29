---
title: Effective Java 2rd
date: 2019-03-28 09:01:14
tags:
- java
- book
- 书
categories:
- Java
---

# 前言

浏览《Effective Java 第二版》中文版，资源[Gitbook Effective Java 2rd](https://zhengyq.gitbooks.io/effective-java-2/content/chapter1.html)

---

第9条：覆盖`equals`时总要覆盖`hashCode`

> *在每个覆盖了equals方法的类中，也必须覆盖hashCode方法*。如果不这样做的话，就会违反`Object.hashcode`的通用约定，从而导致该类无法结合所有基于散列的集合一起正常工作，这样的集合包括`HashMap`、`HashSet`和`Hashtable`。



第18条：接口优于抽象类

> *通过对你导出的每个重要接口都提供一个抽象的骨架实现（skeletal implementation）类，把接口和抽象类的优点结合起来*。接口的作用仍然是定义类型，但是骨架实现接管了所有与接口实现相关的工作。