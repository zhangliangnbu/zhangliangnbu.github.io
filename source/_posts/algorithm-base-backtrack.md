---
title: 算法-回溯
date: 2020-01-24 15:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 简介

**解决一个回溯问题，实际上就是一个决策树的遍历过程**。你只需要思考 3 个问题：

- 路径：也就是已经做出的选择。
- 选择列表：也就是你当前可以做的选择。
- 结束条件：也就是到达决策树底层，无法再做选择的条件。

时空复杂度：时间复杂度都不可能低于 O(N!)，因为穷举整棵决策树是无法避免的。这也是回溯算法的一个特点，不像动态规划存在重叠子问题可以优化，回溯算法就是纯暴力穷举，复杂度一般都很高。

适用场景：一般用于求具体路径！

<!--more-->

# 代码框架

写 backtrack 函数时，需要维护走过的「路径」和当前可以做的「选择列表」，当触发「结束条件」时，将「路径」记入结果集。

```java
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

# 经典题目

[78.子集（中等）](https://leetcode-cn.com/problems/subsets)

[46.全排列（中等）](https://leetcode-cn.com/problems/permutations)

[77.组合（中等）](https://leetcode-cn.com/problems/combinations)

[698. 划分为k个相等的子集（中等）](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets/)

[797. 所有可能的路径](https://leetcode-cn.com/problems/all-paths-from-source-to-target/)

[51.N皇后（困难）](https://leetcode-cn.com/problems/n-queens)

[37.解数独（困难）](https://leetcode-cn.com/problems/sudoku-solver/)

# 简单示例-全排列

给定一个不含重复数字的数组 nums ，返回其 所有可能的全排列 。你可以 按任意顺序 返回答案。

```java
// 结果集
private List<List<Integer>> lists = new ArrayList<>();

public List<List<Integer>> permute(int[] nums) {
		// 选择列表即nums本身

		// 路径列表
    LinkedList<Integer> tracks = new LinkedList<>();
		// 回溯
    backtrack(nums, tracks);
    return lists;
}

public void backtrack(int[] nums, LinkedList<Integer> tracks) {
		// 选择终止条件
    if (nums.length == tracks.size()) {
        lists.add(new ArrayList<>(tracks));
        return;
    }

    for (int num : nums) {
				// 对选择列表进行剪枝
        if (tracks.contains(num)) {
            continue;
        }
				// 选择
        tracks.add(num);
        backtrack(nums, tracks);
				// 撤销选择
        tracks.removeLast();
    }
}
```

# 扩展

某种程度上说，动态规划的暴力求解阶段就是回溯算法。只是有的问题具有重叠子问题性质，可以用 dp table 或者备忘录优化，将递归树大幅剪枝，这就变成了动态规划。

用回溯法解题的一般步骤：

（1）针对所给问题，确定问题的解空间：首先应明确定义问题的解空间，问题的解空间应至少包含问题的一个（最优）解。

（2）确定结点的扩展搜索规则

（3）以深度优先方式搜索解空间，并在搜索过程中用剪枝函数避免无效搜索。

# 参考

- [决策树 多叉树的遍历并处理](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/hui-su-suan-fa-xiang-jie-xiu-ding-ban)