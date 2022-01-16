---
title: 数据结构和算法-并查集
date: 2020-01-19 15:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 简介

并查集（英文：Disjoint-set data structure，直译为不交集数据结构），用于处理一些不交集（Disjoint sets，一系列没有重复元素的集合）的合并及查询问题。并查集支持如下操作：

- 查询：查询某个元素属于哪个集合，通常是返回集合内的一个“代表元素”（一般为根节点）。这个操作是为了判断两个元素是否在同一个集合之中。
- 合并：将两个集合合并为一个。
- 添加：添加一个新集合，其中有一个新元素。添加操作不如查询和合并操作重要，常常被忽略

由于支持查询和合并这两种操作，并查集在英文中也被称为联合-查找数据结构（Union-find data structure）或者合并-查找集合（Merge-find set）。

一般来说，“并查集”特指其中最常见的一种实现：不交集森林（Disjoint-set forest）。经过优化的不交集森林有线性的空间复杂度，以及接近常数的单次操作平均时间复杂度，是效率最高的常见数据结构之一。

并查集是用于计算最小生成树的克鲁斯克尔算法中的关键。由于最小生成树在网络路由等场景下十分重要，并查集也得到了广泛的引用。此外，并查集在符号计算，寄存器分配等方面也有应用。

具体适用场景：凡是涉及到元素的分组管理问题，都可以考虑使用并查集进行维护。

- 维护无向图的连通性：支持判断两个点是否在同一连通块内。
- 判断增加一条边是否会产生环：用在求解最小生成树的Kruskal算法里。

# 不交集森林

## **表示**

不交集森林把每一个集合以一棵树表示，每一个节点即是一个元素。节点保存着到它的父节点的[引用](https://zh.wikipedia.org/wiki/引用)，树的根节点则保存一个空引用或者到自身的引用或者其他无效值，以表示自身为根节点。

## **添加（初始化）**

添加操作`MakeSet(x)`添加一个元素`x`，这个元素单独属于一个仅有它自己的集合。在不交集森林中，添加操作仅需将元素标记为根节点。用伪代码表示如下：

```arduino
function MakeSet(x)
     x.parent := x
end function
```

在经过优化的不交集森林中，添加操作还会初始化一些有关节点的信息，例如集合的大小或者秩。

## **查询（带路径压缩优化）**

在不交集森林中，每个集合的代表即是集合的根节点。查询操作`Find(x)`从`x`开始，根据节点到父节点的引用向根行进，直到找到根节点。用伪代码表示如下：

```arduino
function Find(x)
		if x.parent = x then
				return x
		else 
				return Find(x.parent)
		end if
end function
```

**路径压缩优化**

在集合很大或者树很不平衡时，上述代码的效率很差，最坏情况下（树退化成一条链时），单次查询的时间复杂度高达O(n)。一个常见的优化是路径压缩：在查询时，把被查询的节点到根节点的路径上的所有节点的父节点设置为根结点，从而减小树的高度。也就是说，在向上查询的同时，把在路径上的每个节点都直接连接到根上，以后查询时就能直接查询到根节点。注意，这仅仅只是压缩了某一个路径上的节点。用伪代码表示如下：

```arduino
function Find(x)
     if x.parent = x then
         return x
     else
         x.parent := Find(x.parent)
         return x.parent
     end if
 end function
```

## 合并（按大小或按秩合并）

合并操作`Union(x, y)`把元素`x`所在的集合与元素`y`所在的集合合并为一个。合并操作首先找出节点`x`与节点`y`对应的两个根节点，如果两个根节点其实是同一个，则说明元素`x`与元素`y`已经位于同一个集合中，否则，使其中一个根节点成为另一个的父节点。

```arduino
// 普通合并
function Union(x, y)
     xRoot := Find(x)
     yRoot := Find(y)
     
     if xRoot ≠ yRoot then
         xRoot.parent := yRoot
     end if
 end function
```

**按秩合并优化**

上述代码的问题在于，可能会使得树不平衡，增大树的深度，从而增加查询的耗时。一个控制树的深度的办法是，在合并时，比较两棵树的大小，较大的一棵树的根节点成为合并后的树的根节点，较小的一棵树的根节点则成为前者的子节点。判断树的大小有两种常用的方法，一个是以树中元素的数量作为树的大小，这被称为**按大小合并**。另一种做法则是使用“秩”来比较树的大小。”秩“的定义如下：

- 只有根节点的树（即只有一个元素的集合），秩为0；
- 当两棵秩不同的树合并后，新的树的秩为原来两棵树的秩的较大者；
- 当两棵秩相同的树合并后，新的树的秩为原来的树的秩加一。

容易发现，在没有路径压缩优化时，树的秩等于树的深度减一。在有路径压缩优化时，树的秩仍然能反映出树的深度和大小。在合并时根据两棵树的秩的大小，决定新的根节点，这被称作**按秩合并**。用伪代码表示如下：

```arduino
function MakeSet(x)
     x.parent := x
     x.rank := 0
 end function
 
 function Union(x, y)
     xRoot := Find(x)
     yRoot := Find(y)
     
     if xRoot ≠ yRoot then
         if xRoot.rank < yRoot.rank then
             large := yRoot
             small := xRoot
         else
             large := xRoot
             small := yRoot
         end if
         
         small.parent := large
         if large.rank = small.rank then
             large.rank := large.rank + 1
         end if
     end if
 end function
```

只有根节点的rank有意义，非根节点的rank是没有意义的。

### 时间及空间复杂度

**空间复杂度：**容易看出，不交集森林的空间复杂度是$O(n)

空间复杂度：对于同时使用路径压缩和按秩合并优化的不交集森林，每个查询和合并操作的平均时间复杂度仅为*O(\alpha(n))*，$\alpha(n)$是反阿克曼函数。由于阿克曼函数$A$增加极度迅速，所以$\alpha(n)$增长极度缓慢，对于任何在实践中有意义的元素数目*n*，$\alpha(n)$均小于5，因此，也可以粗略地认为，并查集的操作有常数的时间复杂度。

# 示例-省份数量

[547. 省份数量](https://leetcode-cn.com/problems/number-of-provinces/)

有 n 个城市，其中一些彼此相连，另一些没有相连。如果城市 a 与城市 b 直接相连，且城市 b 与城市 c 直接相连，那么城市 a 与城市 c 间接相连。

省份 是一组直接或间接相连的城市，组内不含其他没有相连的城市。

给你一个 n x n 的矩阵 isConnected ，其中 isConnected[i][j] = 1 表示第 i 个城市和第 j 个城市直接相连，而 isConnected[i][j] = 0 表示二者不直接相连。

返回矩阵中 省份 的数量。

```java
class Solution {
    public int findCircleNum(int[][] isConnected) {
        int n = isConnected.length;
        // 初始化
        UnionFind uf = new UnionFind(n);
        // 遍历两个点之间的关系，做处理
        for (int i = 0; i < n; i ++) {
            for (int j = i + 1; j < n; j ++) {
                if (isConnected[i][j] == 1) {
                    uf.union(i, j);
                }
            }
        }

        return uf.size;
    }
}

// 并查集对象
class UnionFind {
    // 每个节点对应的父节点
    int[] parent;
    // 不交集个数
    int size;

    public UnionFind(int n) {
        // 初始化：n个不交集
        parent = new int[n];
        for (int i = 0; i < n; i ++) {
            parent[i] = i;
        }
        size = n;
    }

    // 查找：带路径压缩优化
    public int find(int index) {
        if (parent[index] != index) {
            parent[index] = find(parent[index]);
        }
        return parent[index];
    }

    // 普通合并
    public void union(int p, int q) {
        int pRoot = find(p);
        int qRoot = find(q);
        if (pRoot != qRoot) {
            parent[pRoot] = qRoot;
            size --;
        }
    }
}
```

# 示例：连通网络的操作次数

[1319. 连通网络的操作次数](https://leetcode-cn.com/problems/number-of-operations-to-make-network-connected/)

用以太网线缆将 n 台计算机连接成一个网络，计算机的编号从 0 到 n-1。线缆用 connections 表示，其中 connections[i] = [a, b] 连接了计算机 a 和 b。

网络中的任何一台计算机都可以通过网络直接或者间接访问同一个网络中其他任意一台计算机。

给你这个计算机网络的初始布线 connections，你可以拔开任意两台直连计算机之间的线缆，并用它连接一对未直连的计算机。请你计算并返回使所有计算机都连通所需的最少操作次数。如果不可能，则返回 -1 。

```java
class Solution {
    public int makeConnected(int n, int[][] connections) {
        // 并查集
        // 连接数小于n-1的必定不能链接，返回-1
        // 连接数大约等于n-1的必定能链接，链接操作数等于不交集个数-1
        // 遍历连接数，合并
        int m = connections.length;
        if (m < n - 1) {
            return -1;
        }
        UnionFind nf = new UnionFind(n);
        for (int i = 0; i < m; i ++) {
            nf.union(connections[i][0], connections[i][1]);
        }

        return nf.size - 1;
    }

    
}

class UnionFind {
    // 数组值为父节点
    int[] parent;
    // 秩
    int[] rank;
    // 不交集个数
    int size;

    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i ++) {
            parent[i] = i;
            rank[i] = 0;
        }
        size = n;
    }

    public int find(int x) {
        if (parent[x] == x) {
            return x;
        } else {
            parent[x] = find(parent[x]);
            return parent[x];
        }
    }

    public void union(int x, int y) {
        int xRoot = find(x);
        int yRoot = find(y);
        if (xRoot == yRoot) {
            return;
        }
        if (rank[xRoot] < rank[yRoot]) {
            parent[xRoot] = yRoot;
        } else if (rank[xRoot] > rank[yRoot]){
            parent[yRoot] = xRoot;
        } else if (rank[xRoot] == rank[yRoot]) {
            parent[yRoot] = xRoot;
            rank[xRoot] = rank[xRoot] + 1;
        }
        size --;
    }
}
```

# 相关题目

- [547. 省份数量](https://leetcode-cn.com/problems/number-of-provinces/)
- [959. 由斜杠划分区域](https://leetcode-cn.com/problems/regions-cut-by-slashes/)，二维点格一维话，将点格分为四个部分
- [1319. 连通网络的操作次数](https://leetcode-cn.com/problems/number-of-operations-to-make-network-connected/)
- [1202. 交换字符串中的元素](https://leetcode-cn.com/problems/smallest-string-with-swaps/)，并查集，优先队列
- [684. 冗余连接](https://leetcode-cn.com/problems/redundant-connection/)
- [947. 移除最多的同行或同列石头](https://leetcode-cn.com/problems/most-stones-removed-with-same-row-or-column/)，按点合并，或按边合并（一个点代表两个边）
- [721. 账户合并](https://leetcode-cn.com/problems/accounts-merge/)，字符串映射为整数，再做处理
- [399. 除法求值](https://leetcode-cn.com/problems/evaluate-division/)，带权并查集
- [765. 情侣牵手](https://leetcode-cn.com/problems/couples-holding-hands/)，理解题意，逐步推导
- [803. 打砖块](https://leetcode-cn.com/problems/bricks-falling-when-hit/)
- [778. 水位上升的泳池中游泳](https://leetcode-cn.com/problems/swim-in-rising-water/)

# 参考

- [维基百科-并查集](https://zh.wikipedia.org/zh-cn/并查集)
- [三方笔记-并查集](https://zhuanlan.zhihu.com/p/93647900)
