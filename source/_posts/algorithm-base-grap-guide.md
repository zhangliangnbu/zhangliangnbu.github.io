---
title: 数据结构和算法-图论基础
date: 2020-01-21 15:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 简介

图（Grap）是用来对对象之间的成对关系建模的数学结构，由"节点"或"顶点"(Vertex）以及连接这些顶点的"边"（Edge）组成。

图包含一个有限（可能是可变的）的集合作为节点集合，以及一个无序对（对应无向图）或有序对（对应有向图）的集合作为边（有向图中也称作弧）的集合。

图的数据结构还可能包含和每条边相关联的数值（edge value），例如一个标号或一个数值（即权重，weight；表示花费、容量、长度等），表示加权图。

**图的分类**

- 无向图和有向图。
- 无权图和有权图。每条边是否有相关联的数值，有的话就是有权图，否则就是无权图。

**图的连通性**

在图论中，连通图基于连通的概念。在一个无向图 G 中，若从顶点 i 到顶点 j 有路径相连（当然从j到i也一定有路径），则称 i 和 j 是连通的。如果 G 是有向图，那么连接i和j的路径中所有的边都必须同向。如果图中任意两点都是连通的，那么图被称作连通图。如果此图是有向图，则称为强连通图（注意：需要双向都有路径）。图的连通性是图的基本性质。

**完全图：**完全是一个简单的无向图，其中每对不同的顶点之间都恰连有一条边相连。

**自环边：**一条边的起点终点是一个点。

**平行边：**两个顶点之间存在多条边相连接。

**节点的度**

无向图中，节点的度（英语：degree或valency）表示该节点作为一个端点的边的数目。

有向图中，分别为节点的入度和出度

- 入度：（英语：in-degree）是指终点为该节点的边的数量。
- 出度：（英语：out-degree）是指起点为该节点的边的数量。

**路径和回路**

- 路径。无论是无向图还是有向图，从一个顶点到另一顶点途径的所有顶点组成的序列（包含这两个顶点），称为一条路径。若路径中各顶点都不重复，此路径又被称为"简单路径"。
- 回路或环。如果路径中第一个顶点和最后一个顶点相同，则此路径称为"回路"（或"环"）。同样，若回路中的顶点互不重复，此回路被称为"简单回路"（或简单环）。
- 路径长度。是指一条路径上经过的边的数量。

<!--more-->

# 图的存储结构

- 邻接表。节点存储为记录或对象，且为每个节点创建一个列表。这些列表可以按节点存储其余的信息；例如，若每条边也是一个对象，则将边存储到边起点的列表上，并将边的终点存储在边这个的对象本身。
- 邻接矩阵。一个二维矩阵，其中行与列分别表示边的起点和终点。顶点上的值存储在外部。矩阵中可以存储边的值。

邻接表在稀疏图上比较有效率。邻接矩阵则常在图比较稠密的时候使用，判断标准一般为边的数量|E |接近于节点的数量的平方；邻接矩阵也在查找两节点邻接情况较为频繁时使用。

# 图的遍历

图的遍历方法有深度优先搜索法DFS和广度(宽度)优先搜索法BFS。BFS 和 DFS 常用来解决树形数据结构（包括图）的问题。核心思想是把一些问题抽象成图，从一个点开始，向四周或下级开始扩散。

- BFS，广度优先搜索。使用迭代和队列，根据树的层级从上往下遍历。常用于求最短路径、最少步数之类，较DFS耗时少，但耗空间。一般实现：从一个顶点开始，尝试访问尽可能靠近它的顶点。本质上这种搜索在图上是逐层移动的，首先检查最靠近第一个顶点的层，再逐渐向下移动到离起始顶点最远的层。
- DFS，深度优先搜索。使用递归，可使用先序、中序或后序遍历等。BFS能做的，DFS也能完成，可记录路径。一般较BFS耗时，但省空间，逻辑也比较清晰。一般实现：访问一个没有访问过的顶点，将它标记为已访问，再递归地去访问在起始点的邻接表中其他没有访问过的顶点。深度优先遍历可以用来求连通分量。

解决问题的核心：

- 节点其实是一种状态，节点的扩散是状态的演变。
- 如何确定节点（或状态），节点之间如何连接或转移？
- 如何简化状态的存储？比如用字符串代替数组等。

## 框架

BFS 框架，求最短路径或步数等：

```java
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路

    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```

DFS框架，多叉树的遍历，如果是图则增加备忘录数组：

```java
void dfs(Node node, Set<Node> visited) {
	if (visited.contains(node)) {
		return;
	}

	// 对node 进行处理 前序
	for (Node child : node.children()) {
		dfs(child)
	}
	// 对node 进行处理 后序
}
```

## 题目-BFS

- [111.二叉树的最小深度（简单）](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree)
- [面试题 04.03. 特定深度节点链表](https://leetcode-cn.com/problems/list-of-depth-lcci/)
- [752.打开转盘锁（中等）](https://leetcode-cn.com/problems/open-the-lock) 将状态作为图的节点，遍历状态
- [433. 最小基因变化](https://leetcode-cn.com/problems/minimum-genetic-mutation/)（中等）
- [773.滑动谜题（困难）](https://leetcode-cn.com/problems/sliding-puzzle/)，将状态作为图的节点，遍历状态
- [127. 单词接龙](https://leetcode-cn.com/problems/word-ladder/)
- [126. 单词接龙 II](https://leetcode-cn.com/problems/word-ladder-ii/)，需要返回具体路径！BFS确定步数、Map记录演变，DFS回溯求具体路径。
- [1284. 转化为全零矩阵的最少反转次数](https://leetcode-cn.com/problems/minimum-number-of-flips-to-convert-binary-matrix-to-zero-matrix/)

## 题目-DFS 和 BFS

- [695. 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)，DFS 或 BFS
- [547. 省份数量](https://leetcode-cn.com/problems/number-of-provinces/)，DFS或BFS
- [130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)， DFS或BFS
- [690. 员工的重要性](https://leetcode-cn.com/problems/employee-importance/)，DFS 或 BFS
- [365. 水壶问题](https://leetcode-cn.com/problems/water-and-jug-problem/)， DFS 或 BFS
- [面试题 16.19. 水域大小](https://leetcode-cn.com/problems/pond-sizes-lcci/)
- *[329. 矩阵中的最长递增路径](https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix/)* （困难），增加备忘录减少耗时。可逆向求出具体路径。
- [778. 水位上升的泳池中游泳](https://leetcode-cn.com/problems/swim-in-rising-water/)

## 求具体路径：单词接龙 二

```java
public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList) {
	// BFS 求步数和记录 + DFS 求具体路径
	Queue<String> queue = new LinkedList<>();
	Set<String> visited = new HashSet<>();
	Set<String> bank = new HashSet<>(wordList);
	// 辅助，单词（key）从哪些单词（value）演变而来，用于DFS求具体路径
	Map<String, HashSet<String>> from = new HashMap<>();
	// 辅助from，演变到单词（key）时的最小步数
	Map<String, Integer> steps = new HashMap<>();
	boolean found = false;
	int step = 1;

	// 异常条件
	if (!bank.contains(endWord)) {
		return new ArrayList<>();
	}

	int len = beginWord.length();
	queue.offer(beginWord);
	visited.add(beginWord);
	steps.put(beginWord, 0);

	while(!queue.isEmpty()) {
		int size = queue.size();
		for (int i = 0; i < size; i ++) {
			// 当前单词
			String cur = queue.poll();

			if (cur.equals(endWord)) {
				found = true;
				break;
			}

			for (int j = 0; j < len; j ++) {
				char src = cur.charAt(j);
				for (char target = 'a'; target <= 'z'; target ++) {
					if (target == src) {
						continue;
					}
					// 下一个单词
					String chg = changChar(cur, j, target);

					// 第一次出现 或者 同一层次，相同的单词
					if (!steps.containsKey(chg) || (steps.containsKey(chg) && steps.get(chg) == step)) {
						if (!from.containsKey(chg)) {
							from.put(chg, new HashSet<>());
						}
						from.get(chg).add(cur);
					}
					if (!steps.containsKey(chg)) {
						steps.put(chg, step);
					}

					if (!visited.contains(chg) && bank.contains(chg)) {
						queue.offer(chg);
						visited.add(chg);
					}
				}
			}
		}

		step ++;
		if (found) {
			break;
		}
	}

	// DFS 从endWord -> startWord
	List<List<String>> res = new ArrayList<>();
	if (found) {
	    Deque<String> path = new ArrayDeque<>();
	  	path.offerFirst(endWord);
	  	dfs(res, path, from, beginWord, endWord);	
	}

	return res;
}

public String changChar(String str, int index, char c) {
	char[] arr = str.toCharArray();
	arr[index] = c;
	return new String(arr);
}

public void dfs(List<List<String>> res, Deque<String> path, Map<String, HashSet<String>> select, 
	String startWord, String cur) {
	if (cur.equals(startWord)) {
		res.add(new ArrayList<>(path));
		return;
	}

	for(String word : select.get(cur)) {
		path.offerFirst(word);
		dfs(res, path, select, startWord, word);
		path.pollFirst();
	}
}
```

# 其他应用和算法

**拓扑排序（Topological Sorting）**

是一个有向无环图（DAG, Directed Acyclic Graph）的所有顶点的线性序列。

**最短路径**

对于无权图，最短路径就是起点到终点的最短边数。一般用BFS计算，借助“剥洋葱”特性，看从起点到终点需要剥几个洋葱圈，剥的层数即是最短路径长度。DFS也可以求，但耗时。

对于有权图，最短路径就是找图中连接两个顶点所有边权重和最小的路径。

- 如果给定起点，则是单源最短路径，即从一固定起点到任意终点的最短路径。相关算法有：`Dijkstra`算法。
- 如果起点不确定，则是多源最短路径，即求任意两个顶点的最短路径。相关算法有：`Dijkstra`算法（将Djikstra算法在每个顶点调用一遍）、Floyd算法。

**最小生成树**

TODO

# 参考

- [维基百科-图 (数据结构)](https://zh.wikipedia.org/wiki/图_(数据结构))
- [（数据结构）图的存储结构完全攻略](http://data.biancheng.net/view/200.html)