---
title: 数据结构和算法-二叉树
date: 2020-01-18 15:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

树形结构包括：线段树、自平衡二叉查找树、B树、二叉树、AA树、AVL树、红黑树、平衡树、伸展树、二叉查找树、堆、二叉堆、左偏树、二项堆、斐波那契堆、R树、R*树、R+树、Hilbert R树、前缀树、哈希树、图等等。本章介绍最基本的二叉树，其他所有的树都能转变为二叉树处理。

<!--more-->

# 二叉树的基本操作

四种搜索遍历的递归和迭代写法

- 前序遍历，根左右。自顶而下，深度优先（DFS）。
- 后序遍历，左右根。自下而上，深度优先（DFS）。
- 中序遍历，左根右。深度优先（DFS）。
- 层次遍历。广度优先（BFS）

> 前中后基于遍历时根的顺序而言。

## 前序遍历

```java
// 递归
private void recursion(TreeNode root, List<Integer> ts) {
    if (root == null) {
        return;
    }
    ts.add(root.val);
    recursion(root.left, ts);
    recursion(root.right, ts);
}

// 迭代
public List<Integer> traversalByIteration(TreeNode root) {
    // 迭代 使用栈
    List<Integer> ts = new ArrayList<>();
    Stack<TreeNode> stack = new Stack<>();
    if (root != null) {
        stack.push(root);
    }
    while (!stack.empty()) {
        TreeNode r = stack.pop();
        // 对节点进行处理
        ts.add(r.val);
        if (r.right != null) {
            stack.push(r.right);
        }
        if (r.left != null) {
            stack.push(r.left);
        }
    }
    return ts;
}
```

## 中序遍历

```java
// 递归
private void recursion(TreeNode root, List<Integer> ts) {
    if (root == null) {
        return;
    }
    recursion(root.left, ts);
    ts.add(root.val);
    recursion(root.right, ts);
}

// 迭代
public List<Integer> traversalByIteration(TreeNode root) {
    // 迭代 使用栈
    List<Integer> ts = new ArrayList<>();
    Stack<TreeNode> stack = new Stack<>();
    while (root != null || !stack.empty()) {
        while (root != null) {
            stack.push(root);
            root = root.left;
        }
        root = stack.pop();
        ts.add(root.val);
        root = root.right;
    }
    return ts;
}
```

## 后序遍历

```java
// 递归
private void recursion(TreeNode root, List<Integer> ts) {
    if (root == null) {
        return;
    }
    recursion(root.left, ts);
    recursion(root.right, ts);
    ts.add(root.val);
}

// 迭代
public List<Integer> traversalByIteration(TreeNode root) {
    List<Integer> ts = new ArrayList<>();
    Stack<TreeNode> stack = new Stack<>();
    if (root != null) {
        stack.add(root);
    }
    while (!stack.empty()) {
        root = stack.pop();
        ts.add(0, root.val);
        if (root.left != null) {
            stack.add(root.left);
        }
        if (root.right != null) {
            stack.add(root.right);
        }
    }
    return ts;
}
```

## 层级遍历

```java
// 递归
private void recursion(TreeNode root, int level, List<List<Integer>> ts) {
    if (root == null) {
        return;
    }
    if (level > ts.size()) {
        ts.add(new ArrayList<>());
    }
    ts.get(level - 1).add(root.val);
    recursion(root.left, level + 1, ts);
    recursion(root.right, level + 1, ts);
}

// 迭代
public List<List<Integer>> traversalByIteration(TreeNode root) {
    // 迭代 使用队列
    List<List<Integer>> ts = new ArrayList<>();
    Queue<TreeNode> queue = new LinkedList<>();
    if (root != null) {
        queue.offer(root);
    }
    List<Integer> ss = new ArrayList<>();
    // 记录当前ss长度上限
    int ssLen = 1;
    while (!queue.isEmpty()) {
        root = queue.poll();
        ss.add(root.val);
        if (root.left != null) {
            queue.offer(root.left);
        }
        if (root.right != null) {
            queue.offer(root.right);
        }

        // 当前需要的长度
        if (ssLen == ss.size()) {
            ts.add(ss);
            ss = new ArrayList<>();
            ssLen = queue.size();
        }
    }
    return ts;
}
```

## 自顶向下和自顶向上

**自顶向下**

“自顶向下” 意味着在每个递归层级，我们将首先访问节点来计算一些值，并在递归调用函数时将这些值传递到子节点。 所以 “自顶向下” 的解决方案可以被认为是一种前序遍历。 具体来说，递归函数 top_down(root, params) 的原理是这样的：

```java
1. return specific value for null node
2. update the answer if needed     // anwer <-- params
3. left_ans = top_down(root.left, left_params)  // left_params <-- root.val, params
4. right_ans = top_down(root.right, right_params) // right_params <-- root.val, params
5. return the answer if needed       // answer <-- left_ans, right_ans
```

**自底向上**

“自底向上” 是另一种递归方法。 在每个递归层次上，我们首先对所有子节点递归地调用函数，然后根据返回值和根节点本身的值得到答案。 这个过程可以看作是后序遍历的一种。 通常， “自底向上” 的递归函数 bottom_up(root) 为如下所示：

```java
1. return specific value for null node
2. left_ans = bottom_up(root.left)    // call function recursively for left child
3. right_ans = bottom_up(root.right)    // call function recursively for right child
4. return answers   // answer <-- left_ans, right_ans, root.val
```

小结

自顶向下：

- 你能确定一些参数，并从该节点自身解决出发寻找答案。
- 你可以使用这些参数和节点本身的值来决定什么应该是传递给它子节点的参数。

自底向上：对于树中的任意一个节点，如果你知道它子节点的答案，你能计算出该节点的答案。

# **二叉搜索树（二叉查找树）**

概念：所有left.val <= root.val <= 所有right.val

一般查找操作：

```java
void BST(TreeNode root, int target) {
    if (root.val == target)
        // 找到目标，做点什么...
    if (root.val < target)
        BST(root.right, target);
    if (root.val > target)
        BST(root.left, target);
}
```

平衡二叉查找树：AVL树，高度平衡的二叉搜索树

# **红黑树**

红黑树是一种特化的AVL树（平衡二叉树），都是在进行插入和删除操作时通过特定操作保持二叉查找树的平衡，从而获得较高的查找性能。

它虽然是复杂的，但它的最坏情况运行时间也是非常良好的，并且在实践中是高效的： 它可以在O(log n)时间内做查找，插入和删除，这里的n 是树中元素的数目。

红黑树是每个节点都带有颜色属性的二叉查找树，颜色或红色或黑色。属性如下：

- 性质1. 节点是红色或黑色。
- 性质2. 根节点是黑色。
- 性质3.所有叶子都是黑色。（叶子是NIL节点）
- 性质4. 每个红色节点的两个子节点都是黑色。（从每个叶子到根的所有路径上不能有两个连续的红色节点）
- 性质5.从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

这些约束强制了红黑树的关键性质: **从根到叶子的最长的可能路径不多于最短的可能路径的两倍长**。结果是这个树大致上是平衡的。因为操作比如插入、删除和查找某个值的最坏情况时间都要求与树的高度成比例，这个在高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。

- 参考：https://www.cnblogs.com/skywang12345/p/3245399.html

# 二叉堆

简介：有最大堆和最小堆，最大堆即父节点大于子节点，最小堆相反。

应用于排序算法，堆排序，用数组表示的二叉树，且左子树排完后再排右子树。

在最大堆中，父节点的值比每一个子节点的值都要大。在最小堆中，父节点的值比每一个子节点的值都要小。这就是所谓的“堆属性”，并且这个属性对堆中的每一个节点都成立。

每个节点的父和子的索引公式：

```java
parent(i) = floor((i - 1)/2)
left(i)   = 2i + 1
right(i)  = 2i + 2
```

堆的高度：如果一个堆有 n 个节点，那么它的高度是 h = floor(log2(n))。

叶节点总是位于数组的 floor(n/2) 和 n-1 之间。

有两个原始操作用于保证插入或删除节点以后堆是一个有效的最大堆或者最小堆：

- `shiftUp()`: 如果一个节点比它的父节点大（最大堆）或者小（最小堆），那么需要将它同父节点交换位置。这样是这个节点在数组的位置上升。
- `shiftDown()`: 如果一个节点比它的子节点小（最大堆）或者大（最小堆），那么需要将它向下移动。这个操作也称作“堆化（heapify）”。

参考：

- https://github.com/raywenderlich/swift-algorithm-club/tree/master/Heap
- https://www.jianshu.com/p/6b526aa481b1

# **实践**

## 子树匹配问题

思路：从某个根节点开始匹配，计算是否匹配目标子树。

相关问题

- leetcode 1367. 二叉树中的列表
- leetcode 面试题 04.10. 检查子树
- leetcode 572. 另一个树的子树

## 二叉树构建

确定根节点，自上而下，逐层构建

- leetcode 1008. 前序遍历构造二叉搜索树
- leetcode 106. 从中序与后序遍历序列构造二叉树
- leetcode 105. 从前序与中序遍历序列构造二叉树
- leetcode 面试题 04.02. 最小高度树：中间节点为根节点
- leetcode 108. 将有序数组转换为二叉搜索树
- 从先序遍历还原二叉树：https://leetcode-cn.com/problems/recover-a-tree-from-preorder-traversal/

## 二叉树镜像或翻转

遍历，左右互换即可

- leetcode 226. 翻转二叉树
- leetcode 剑指 Offer 27. 二叉树的镜像

## 最近公共祖先

针对单个根节点分析三种情况，自下而上遍历即可

- 剑指 Offer 68 - II. 二叉树的最近公共祖先
- 剑指 Offer 68 - I. 二叉搜索树的最近公共祖先：https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/
- 面试题 04.08. 首个共同祖先：https://leetcode-cn.com/problems/first-common-ancestor-lcci/

## 路径和

必要时，使用额外栈或存储父链接

- 路径总和：https://leetcode-cn.com/problems/path-sum/
- 路径总和 II：https://leetcode-cn.com/problems/path-sum-ii/
- 路径总和 III：https://leetcode-cn.com/problems/path-sum-iii/
- 二叉树中的最大路径和：https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/

## 其他

- [211. 添加与搜索单词 - 数据结构设计](https://leetcode-cn.com/problems/design-add-and-search-words-data-structure/)，构造字典树
