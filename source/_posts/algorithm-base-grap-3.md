---
title: 数据结构和算法-图论的最短路径算法
date: 2020-01-21 17:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 简介

**最短路径**问题是[图论](https://zh.wikipedia.org/wiki/图论)研究中的一个经典算法问题，旨在寻找图（由结点和路径组成的）中两结点之间的最短路径。算法具体的形式包括：

- **确定起点的最短路径问题** - 也叫单源最短路问题，即已知起始结点，求最短路径的问题。在边权非负时适合使用[Dijkstra算法](https://zh.wikipedia.org/wiki/Dijkstra算法)，若边权为负时则适合使用[Bellman-ford算法](https://zh.wikipedia.org/wiki/Bellman-ford)或者[SPFA算法](https://zh.wikipedia.org/wiki/SPFA算法)。
- **确定终点的最短路径问题** - 与确定起点的问题相反，该问题是已知终结结点，求最短路径的问题。在[无向图](https://zh.wikipedia.org/wiki/無向圖)中该问题与确定起点的问题完全等同，在[有向图](https://zh.wikipedia.org/wiki/有向图)中该问题等同于把所有路径方向反转的确定起点的问题。
- **确定起点终点的最短路径问题** - 即已知起点和终点，求两结点之间的最短路径。
- **全局最短路径问题** - 也叫多源最短路问题，求图中所有的最短路径。适合使用[Floyd-Warshall算法](https://zh.wikipedia.org/wiki/Floyd-Warshall算法)。

用于解决最短路径问题的算法被称做“最短路径算法”，有时被简称作“路径算法”。最常用的路径算法有：

- [Dijkstra算法](https://zh.wikipedia.org/wiki/Dijkstra算法)
- [A*算法](https://zh.wikipedia.org/wiki/A星算法)
- [Bellman-Ford算法](https://zh.wikipedia.org/wiki/Bellman-Ford算法)
- [SPFA算法](https://zh.wikipedia.org/wiki/SPFA算法)（Bellman-Ford算法的改进版本）
- [Floyd-Warshall算法](https://zh.wikipedia.org/wiki/Floyd-Warshall算法)
- [Johnson算法](https://zh.wikipedia.org/w/index.php?title=Johnson算法&action=edit&redlink=1)
- [Bi-Direction BFS算法](https://zh.wikipedia.org/w/index.php?title=Bi-Direction_BFS算法&action=edit&redlink=1)

# Dijkstra算法

戴克斯特拉算法（英语：Dijkstra's algorithm），又译迪杰斯特拉算法，亦可不音译而称为Dijkstra算法，使用类似广度优先搜索的方法解决赋权图的单源最短路径问题。

戴克斯特拉的原始版本仅适用于找到两个顶点之间的最短路径，后来更常见的变体固定了一个顶点作为源结点然后找到该顶点到图中所有其它结点的最短路径，产生一个最短路径树。

该算法解决了图$G=(V,E)$上带权的单源最短路径问题。具体来说，戴克斯特拉算法设置了一顶点集合S，在集合S中所有的顶点与源点s之间的最终最短路径权值均已确定。算法反复选择最短路径估计最小的点u（属于集合V-S），并将u加入S中。该算法常用于路由算法或者作为其他图算法的一个子模块。举例来说，如果图中的顶点表示城市，而边上的权重表示城市间开车行经的距离，该算法可以用来找到两个城市之间的最短路径。

应当注意，绝大多数的戴克斯特拉算法不能有效处理带有负权边的图。

## 算法描述

将所有节点分成两类：已确定从起点到当前点的最短路长度的节点，以及未确定从起点到当前点的最短路长度的节点（下面简称「未确定节点」和「已确定节点」）。

每次从「未确定节点」中取一个与起点距离最短的点，将它归类为「已确定节点」，并用它「更新」从起点到其他所有「未确定节点」的距离。直到所有点都被归类为「已确定节点」。

用节点 A「更新」节点 B 的意思是，用起点到节点 A 的最短路长度加上从节点 A 到节点 B 的边的长度，去比较起点到节点 B 的最短路长度，如果前者小于后者，就用前者更新后者。这种操作也被叫做「松弛」。

```arduino
function Dijkstra(G, w, s)
		// 实际上的操作是将每个除原点外的顶点置为无穷大
		// d[v] = INF, d[0] = 0
		INITIALIZE-SINGLE-SOURCE(G, s)
		S <- 空集
		Q <- s  // Q 是顶点V的优先队列，以顶点的最短路径估计排序
		while(Q != 空集)
				do u <- EXTRACT - MIN(Q)  // 取出最小点
				S <- u
				for each vertex v 属于 Adj[u]
						do RELAX(u, v, w)     // 松弛成功的结点会被加入到队列中
```

## 示例-网络延迟时间

[743. 网络延迟时间](https://leetcode-cn.com/problems/network-delay-time/)

有 n 个网络节点，标记为 1 到 n。

给你一个列表 times，表示信号经过 有向 边的传递时间。 times[i] = (ui, vi, wi)，其中 ui 是源节点，vi 是目标节点， wi 是一个信号从源节点传递到目标节点的时间。

现在，从某个节点 K 发出一个信号。需要多久才能使所有节点都收到信号？如果不能使所有节点收到信号，返回 -1 。

```groovy
public int networkDelayTime(int[][] times, int n, int k) {
		// 求出最长路径即可
    // 距离为INF表示不可达，因为有相加的操作，故需要除2操作
    int INF = Integer.MAX_VALUE / 2;
    // 邻接矩阵
    int[][] g = new int[n][n];
    for (int i = 0; i < n; i ++) {
        Arrays.fill(g[i], INF);
    }
    for (int[] time : times) {
        int x = time[0] - 1, y = time[1] - 1;
        g[x][y] = time[2];
    }
    // 当前点到源点的距离
    int[] dist = new int[n];
    Arrays.fill(dist, INF);
    dist[k-1] = 0;
    // 已确定
    boolean[] used = new boolean[n];
    // 遍历n次，每次找到一个距离最短的点
    for (int i = 0; i < n; i ++) {
        // 找到未确定点里的距离最短顶点
        int x = -1;
        for (int y = 0; y < n; y ++) {
            if (!used[y] && (x == -1 || dist[y] < dist[x])) {
                x = y;
            }
        }
        // 标记最小距离点
        used[x] = true;
        // 松弛操作：更新邻接点
        for (int y = 0; y < n; y ++) {
            dist[y] = Math.min(dist[x] + g[x][y], dist[y]);
        }
    }

    // 求出最大时间
    int max = 0;
    for (int d : dist) {
        max = Math.max(d, max);
    }
    return max == INF ? -1 : max;
}
```

使用优先队列，降低时间复杂度：

```groovy
public int networkDelayTime(int[][] times, int n, int k) {
    // Dijkstra
    // 分为确定点和未确定点，每次去除最小未确定点，添加到确定点里并更新
    int INF = Integer.MAX_VALUE / 2;
    // 邻接表
    ArrayList<int[]>[] g = new ArrayList[n];
    for (int i = 0; i < n; i ++) {
        g[i] = new ArrayList<>();
    }
    for (int[] time : times) {
        int x = time[0] - 1;
        int y = time[1] - 1;
        g[x].add(new int[]{y, time[2]});
    }
    // 与k的距离
    int[] dist = new int[n];
    Arrays.fill(dist, INF);
    dist[k-1] = 0;

    // 未确定点
    PriorityQueue<int[]> pq = new PriorityQueue<>(new Comparator<int[]>() {
        public int compare(int[] a, int[] b) {
            // 路径相等时，索引小的放在前面
            return a[1] == b[1] ? a[0] - b[0] : a[1] - b[1];
        }
    });
    pq.offer(new int[]{k-1, dist[k-1]});
    while(!pq.isEmpty()) {
        int[] p = pq.poll();
        int x = p[0], time = p[1];
        // 已经确定过了 确定过的距离比当前距离小
        if (dist[x] < time) {
            continue;
        }
        // 更新邻接点
        for (int[] adj : g[x]) {
            int y = adj[0], dy = time + adj[1];
            if (dy < dist[y]) {
                pq.offer(new int[]{y, dy});
                dist[y] = dy;
            }
        }
	  }

    // 求出最大值
    int max = 0;
    for (int d : dist) {
        max = Math.max(max, d);
    }
    return max == INF ? -1 : max;
}
```

## 相关题目

- [743. 网络延迟时间](https://leetcode-cn.com/problems/network-delay-time/)
- [1514. 概率最大路径](https://leetcode-cn.com/problems/path-with-maximum-probability/)
- [778. 水位上升的泳池中游泳](https://leetcode-cn.com/problems/swim-in-rising-water/)

# Bellman–Ford算法

贝尔曼-福特算法（英语：Bellman–Ford algorithm），求解单源最短路径问题的一种算法，由理查德·贝尔曼（Richard Bellman） 和 莱斯特·福特 创立的。有时候这种算法也被称为 Moore-Bellman-Ford 算法，因为 Edward F. Moore 也为这个算法的发展做出了贡献。它的原理是对图进行 $|V|-1$次松弛操作，得到所有可能的最短路径。其优于迪科斯彻算法的方面是边的权值可以为负数、实现简单，缺点是时间复杂度过高，高达$O(|V||E|)$。但算法可以进行若干种优化，提高了效率。

## 示例

1. 网络延迟时间 有 n 个网络节点，标记为 1 到 n。 给你一个列表 times，表示信号经过 有向 边的传递时间。 times[i] = (ui, vi, wi)，其中 ui 是源节点，vi 是目标节点， wi 是一个信号从源节点传递到目标节点的时间。 现在，从某个节点 K 发出一个信号。需要多久才能使所有节点都收到信号？如果不能使所有节点收到信号，返回 -1 。

**常规**

```java
public int networkDelayTime(int[][] times, int n, int k) {
    // Bellman-Ford 算法：以点拓展深度，以边拓展广度
    int INF = Integer.MAX_VALUE / 2;
    int[] dist = new int[n];
    Arrays.fill(dist, INF);
    dist[k-1] = 0;

    // 拓展深度
    for (int i = 0; i < n; i ++) {
        // 记录上次的距离
        int[] pre = dist.clone();
        // 拓展广度
        for (int[] time : times) {
            int x = time[0]-1, y = time[1]-1, t = time[2];
            dist[y] = Math.min(pre[x] + t, dist[y]);
        }
    }

    // 求最大值
    int max = 0;
    for (int d : dist) {
        max = Math.max(max, d);
    }

    return max == INF ? -1 : max;
}
```

**队列优化**

```java
public int networkDelayTime(int[][] times, int n, int k) {
        // Bellman-Ford 队列优化：只遍历松弛过的点
        // 初始化
        int INF = Integer.MAX_VALUE / 2;
        // 100个点，可以用邻接矩阵
        int[][] g = new int[n][n];
        for (int i = 0; i < n; i ++) {
            Arrays.fill(g[i], INF);
        }
        for (int[] time : times) {
            int x = time[0] - 1, y = time[1] - 1, w = time[2];
            g[x][y] = w;
        }
        // 最短距离
        int[] dist = new int[n];
        Arrays.fill(dist, INF);
        dist[k-1] = 0;
        // 在队列中的点，无需再次入队
        boolean[] inq = new boolean[n];
        
        // 队列，松弛了且不在队列中的点，入队
        Queue<Integer> q = new LinkedList<>();
        q.offer(k-1);
        inq[k-1] = true;
        while (!q.isEmpty()) {
            int x = q.poll();
            inq[x] = false;
            for (int y = 0; y < n; y ++) {
                if (y == x) continue;
                int dy = g[x][y] + dist[x];
                if (dy < dist[y]) {
                    dist[y] = dy;
                    if (!inq[y]) {
                        q.offer(y);
                        inq[y] = true;
                    }
                }
            }
        }

        int max = -1;
        for (int d : dist) {
            max = Math.max(max, d);
        }
        return max == INF ? -1 : max;
    }
```

# 单源最短路径适用算法

## 无权图

算法：广度优先搜索，时间复杂度O(E+V)

## **有向无环图**

使用拓扑排序算法可以在有权值的DAG中以线性时间O(E+V) 求解单源最短路径问题。

## 无负权图

Dijkstra算法：设置了一顶点集合S，在集合S中所有的顶点与源点s之间的最终最短路径权值均已确定。算法反复选择最短路径估计最小的点u（属于集合V-S），并将u加入S中。

## 正负权图

Bellman-Ford 算法：对图进行 $|V|-1$次松弛操作，得到所有可能的最短路径。以点遍历深度，以线拓展广度。

## 多源最短路径

Floyd-Warshall算法。可以正确处理有向图或负权（但不可存在负权回路）的最短路径问题。

原理：使用动态规划，从i到j只以(1..k)集合中的节点为中间节点的最短路径，等于经过k和不经过k的最短路中的最小值。

伪代码：

```java
let dist be a |V| × |V| array of minimum distances initialized to ∞ (infinity)
 for each vertex v
    dist[v][v] ← 0
 for each edge (u,v)
		dist[u][v] ← w(u,v)  // the weight of the edge (u,v)
 for k from 1 to |V|
    for i from 1 to |V|
       for j from 1 to |V|
          if dist[i][j] > dist[i][k] + dist[k][j] 
             dist[i][j] ← dist[i][k] + dist[k][j]
         end if
```

# 参考

- [【最短路/必背模板】涵盖所有的「存图方式」与「最短路算法（详尽注释）」](https://mp.weixin.qq.com/s?__biz=MzU4NDE3MTEyMA==&mid=2247488007&idx=1&sn=9d0dcfdf475168d26a5a4bd6fcd3505d&chksm=fd9cb918caeb300e1c8844583db5c5318a89e60d8d552747ff8c2256910d32acd9013c93058f&token=754098973&lang=zh_CN#rd)
- [戴克斯特拉算法](https://zh.wikipedia.org/wiki/戴克斯特拉算法)
- [力扣阐述](https://leetcode-cn.com/problems/network-delay-time/solution/wang-luo-yan-chi-shi-jian-by-leetcode-so-6phc/)
- [维基百科](https://zh.wikipedia.org/wiki/贝尔曼-福特算法)