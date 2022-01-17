---
title: 数据结构和算法-图论的拓扑排序
date: 2020-01-21 16:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 简介

在图论中，由一个[有向无环图](https://zh.wikipedia.org/wiki/有向无环图)的顶点组成的序列，当且仅当满足下列条件时，才能称为该[图](https://zh.wikipedia.org/wiki/图)的一个拓扑排序（英语：Topological sorting）：

1. 序列中包含每个顶点，且每个顶点只出现一次；
2. 若A在序列中排在B的前面，则在图中不存在从B到A的[路径](https://zh.wikipedia.org/wiki/路径_(图论))。

当且仅当图中没有定向环时（即有向无环图），才有可能进行拓扑排序。有向无环图与拓扑排序互为充要条件。

两个推论：

- 如果图G存在环，则图G不存在拓扑排序。
- 如果图G是有向无环图，则可能存在多个拓扑排序。

求有向无环图的拓扑排序有两种方法：DFS 和 卡恩算法

<!--more-->

# DFS 算法

**说明**

用一个栈来存储所有已经搜索完成的节点。

> 对于一个节点 u，如果它的所有相邻节点都已经搜索完成，那么在搜索回溯到 u 的时候，u 本身也会变成一个已经搜索完成的节点。这里的「相邻节点」指的是从 u 出发通过一条有向边可以到达的所有节点。

假设我们当前搜索到了节点 u，如果它的所有相邻节点都已经搜索完成，那么这些节点都已经在栈中了，此时我们就可以把 u 入栈。可以发现，如果我们从栈顶往栈底的顺序看，由于 u 处于栈顶的位置，那么 u 出现在所有 u 的相邻节点的前面。因此对于 u 这个节点而言，它是满足拓扑排序的要求的。

这样以来，我们对图进行一遍深度优先搜索。当每个节点进行回溯的时候，我们把该节点放入栈中。最终从栈顶到栈底的序列就是一种拓扑排序。

**算法细节**

对于图中的任意一个节点，它在搜索的过程中有三种状态，即：

- 「未搜索」：我们还没有搜索到这个节点；
- 「搜索中」：我们搜索过这个节点，但还没有回溯到该节点，即该节点还没有入栈，还有相邻的节点没有搜索完成）；
- 「已完成」：我们搜索过并且回溯过这个节点，即该节点已经入栈，并且所有该节点的相邻节点都出现在栈的更底部的位置，满足拓扑排序的要求。

通过上述的三种状态，我们就可以给出使用深度优先搜索得到拓扑排序的算法流程，在每一轮的搜索搜索开始时，我们任取一个「未搜索」的节点开始进行深度优先搜索。

- 我们将当前搜索的节点 u 标记为「搜索中」，遍历该节点的每一个相邻节点 v：
  - 如果 v 为「未搜索」，那么我们开始搜索 v，待搜索完成回溯到 u；
  - 如果 v 为「搜索中」，那么我们就找到了图中的一个环，因此是不存在拓扑排序的；
  - 如果 v 为「已完成」，那么说明 v 已经在栈中了，而 u 还不在栈中，因此 u 无论何时入栈都不会影响到 (u,v) 之前的拓扑关系，因此不用进行任何操作。
- 当 u 的所有相邻节点都为「已完成」时，我们将 u 放入栈中，并将其标记为「已完成」。

在整个深度优先搜索的过程结束后，如果我们没有找到图中的环，那么栈中存储这所有的 n 个节点，从栈顶到栈底的顺序即为一种拓扑排序。

维基百科上的伪代码，这里的代码是先判断 u 而非之后判断 v，本质一样：

```arduino
L ← 包含已排序的元素的列表，目前为空
当图中存在未永久标记的节点时：
    选出任何未永久标记的节点n
    visit(n)
function visit(节点 n)
    如n已有永久标记：
        return
    如n已有临时标记：
        stop   (不是定向无环图)
    将n临时标记
    选出以n为起点的边(n,m)，visit(m)
    重复上一步
    去掉n的临时标记
    将n永久标记
    将n加到L的起始
```

# 卡恩算法

卡恩于1962年提出了该算法。简单来说，假设L是存放结果的列表，先找到那些入度为零的节点，把这些节点放到L中，因为这些节点没有任何的父节点。然后把与这些节点相连的边从图中去掉，再寻找图中的入度为零的节点。对于新找到的这些入度为零的节点来说，他们的父节点已经都在L中了，所以也可以放入L。重复上述操作，直到找不到入度为零的节点。如果此时L中的元素个数和节点总数相同，说明排序完成；如果L中的元素个数和节点总数不同，说明原图中存在环，无法进行拓扑排序。

**算法**

我们使用一个队列来进行广度优先搜索。开始时，所有入度为 0 的节点都被放入队列中，它们就是可以作为拓扑排序最前面的节点，并且它们之间的相对顺序是无关紧要的。

在广度优先搜索的每一步中，我们取出队首的节点 u：

- 我们将 u 放入答案中；
- 我们移除 u 的所有出边，也就是将 u 的所有相邻节点的入度减少 1。如果某个相邻节点 v 的入度变为 0，那么我们就将 v 放入队列中。

在广度优先搜索的过程结束后。如果答案中包含了这 n 个节点，那么我们就找到了一种拓扑排序，否则说明图中存在环，也就不存在拓扑排序了。

```arduino
L ← 包含已排序的元素的列表，目前为空
S ← 入度为零的节点的集合
当 S 非空时：
    将节点n从S移走
    将n加到L尾部
    选出任意起点为n的边e = (n,m)，移除e。如m没有其它入边，则将m加入S。
    重复上一步。
如图中有剩余的边则：
    return error   (图中至少有一个环)
否则： 
    return L   (L为图的拓扑排序)
```

# 示例-拓扑排序

[210. 课程表 II](https://leetcode-cn.com/problems/course-schedule-ii/)

现在你总共有 n 门课需要选，记为 0 到 n-1。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们: [0,1]

给定课程总量以及它们的先决条件，返回你为了学完所有课程所安排的学习顺序。

可能会有多个正确的顺序，你只要返回一种就可以了。如果不可能完成所有课程，返回一个空数组。

DFS算法：

```java
class Solution {
    // 排序 0 表示栈顶
    public int[] ans;
    public int index;
    // 访问状态 0 未搜索，1 搜索中，2 已搜索
    public int[] visited;
    // 邻接表 便于使用
    public List<List<Integer>> edges;
    // 是否为有效的拓扑结构
    public boolean valid = true;

    public int[] findOrder(int numCourses, int[][] prerequisites) {
        ans = new int[numCourses];
        index = numCourses - 1;
        visited = new int[numCourses];
        edges = new ArrayList<>();
        for (int i = 0; i < numCourses; i ++) {
            edges.add(new ArrayList<>());
        }
        for (int[] pre : prerequisites) {
            edges.get(pre[1]).add(pre[0]);
        }

        for (int i = 0; i < numCourses && valid; i ++) {
						dfs(i);
        }

        if (!valid) {
            return new int[0];
        } else {
            return ans;
        }
    }

		public void dfs(int cur) {
        // 已完全遍历过，已经入栈不需要遍历
        // 无效则直接退出
        if (visited[cur] == 2 || !valid) {
            return;
        }
        // 已经处于遍历中，说明又回到了出发地，有环
        if (visited[cur] == 1) {
            valid = false;
            return;
        }
        // 标记临时
        visited[cur] = 1;

        // 遍历所有邻接点
        for (int next : edges.get(cur)) {
            dfs(next);
            if (!valid) {
                return;
            }
        }

        // 成功遍历完，标记
        visited[cur] = 2;
        // 入栈 最先入栈的一定是出度为零的点
        ans[index] = cur;
        index --;
    }
}
```

卡恩算法（BFS）：

```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        // 最终的排序 0表示第一个修的课程
        int[] ans = new int[numCourses];
        int index = 0;
        // 入度
        int[] indeg = new int[numCourses];
        // 邻接表 便于使用
        List<List<Integer>> edges = new ArrayList<>();
        for (int i = 0; i < numCourses; i ++) {
            edges.add(new ArrayList<>());
        }
        for (int[] pre : prerequisites) {
            edges.get(pre[1]).add(pre[0]);
            indeg[pre[0]] ++;
        }
        
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i ++) {
            if (indeg[i] == 0) {
                queue.offer(i);
            }
        }
        while(!queue.isEmpty()) {
            int cur = queue.poll();
            //System.out.println("cur ==" + cur);
            ans[index] = cur;
            index ++;
            for (int next : edges.get(cur)) {
                indeg[next] --;
                if (indeg[next] == 0) {
                    queue.offer(next);
                }
            }
        }

        if (index != numCourses) {
            return new int[0];
        }
        return ans;
    }
}
```

# 相关题目

- [207. 课程表](https://leetcode-cn.com/problems/course-schedule/)
- [210. 课程表 II](https://leetcode-cn.com/problems/course-schedule-ii/)
- [802. 找到最终的安全状态](https://leetcode-cn.com/problems/find-eventual-safe-states/)

# 参考

- [维基百科-有向无环图](https://zh.wikipedia.org/wiki/有向无环图)
- [维基百科-拓扑排序](https://zh.wikipedia.org/wiki/拓撲排序)
- [力扣课表问题详细解答](https://leetcode-cn.com/problems/course-schedule-ii/solution/ke-cheng-biao-ii-by-leetcode-solution/)