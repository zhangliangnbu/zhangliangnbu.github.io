---
title: 算法-动态规划
date: 2020-01-23 15:12:03
tags:
- algorithm
- cs
categories:
- 其他
---

# 介绍

**基本思想和策略**

基本思想与分治法类似，也是将待求解的问题分解为若干个子问题（阶段），按顺序求解子阶段，前一子问题的解，为后一子问题的求解提供了有用的信息。在求解任一子问题时，列出各种可能的局部解，通过决策保留那些有可能达到最优的局部解，丢弃其他局部解。依次解决各子问题，最后一个子问题就是初始问题的解。

由于动态规划解决的问题多数有重叠子问题这个特点，为减少重复计算，对每一个子问题只解一次，将其不同阶段的不同状态保存在一个二维数组中。

与分治法最大的差别是：适合于用动态规划法求解的问题，经分解后得到的子问题往往不是互相独立的（即下一个子阶段的求解是建立在上一个子阶段的解的基础上，进行进一步的求解）。

$$ f(n,m)=max{f(n-1,m), f(n-1,m-w[n])+P(n,m)} $$

**通俗理解**

动态规划即要求最值，求解动态规划的核心问题是穷举！列出所有可能的答案，然后找出最值！

一，穷举；二，利用备忘录聪明的穷举。

- 动态规划的穷举有点特别，因为这类问题存在「重叠子问题」，如果暴力穷举的话效率会极其低下，所以需要「备忘录」或者「DP table」来优化穷举过程，避免不必要的计算。
- 动态规划问题一定会具备「最优子结构」，才能通过子问题的最值得到原问题的最值。
- 动态规划的核心思想就是穷举求最值，但是问题可以千变万化，穷举所有可行解其实并不是一件容易的事，只有列出正确的「状态转移方程」，才能正确地穷举。

核心在于：

- 确定基本例
- 确定状态，状态的参数。
- 确定选择，即达到当前状态有几种方式，从这几种方式中选出最优解。
- 明确 dp 函数/数组的定义。

# 实践

## **轮流选择问题**

状态：先手最优解。具体`*dps[i][j]*`表示区间*`i~j`*先手获得的最大值或差值等。

转移：先手最优即次手最差，$dp[i][j] = sum[i][j] - Math.min(dp[i + 1][j], dp[i][j - 1]);$

情景：

- [leetcode 877. 石子游戏](https://leetcode-cn.com/problems/stone-game/)，限制为两端获取
- [leetcode 486. 预测赢家](https://leetcode-cn.com/problems/predict-the-winner/)，限制为两端获取
- [464. 我能赢吗](https://leetcode-cn.com/problems/can-i-win/)，不限制

示例：

```java
// 示例：预测赢家，状态为先手的分数
public boolean PredictTheWinner(int[] nums) {
        int n = nums.length;
        int[][] dps = new int[n][n];
        int[][] sums = new int[n][n];
        for (int i = n - 1; i >= 0; i --) {
            for (int j = i; j < n; j ++) {
                if (i == j) {
                    dps[i][j] = nums[i];
                    sums[i][j] = nums[i];
                } else {
                    sums[i][j] = sums[i][j-1] + nums[j];
                    dps[i][j] = sums[i][j] - Math.min(dps[i + 1][j], dps[i][j - 1]);
                }
            }
        }
        return dps[0][n-1] >= sums[0][n-1] - dps[0][n-1];
}
```

## 最长**子序列问题**

状态一：*`dp[i]`*表示，以`*i*`结尾的最大长度

- [300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)， *`dp[i]`*表示以`*i*`结尾的最大长度
- [53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)
- [354. 俄罗斯套娃信封问题](https://leetcode-cn.com/problems/russian-doll-envelopes/)

状态二：*`dps[i][j]`*表示，单序列时`\*i~j\*`间最长子序列长度，双序列时长度分别为*`0~i`*和`*0~j*`的序列最长公共子序列长度或其他值。

- [leetcode 516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)，

  $dps[i][j] = max(dps[i][j-1], dps[i+1][j] or dps[i+1][j-1] + 2)$

- [leetcode 1143. 最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

  *dps[i][j] = max(dps[i][j-1], dps[i-1][j] or dps[i-1][j-1] + 1)*

- [583. 两个字符串的删除操作](https://leetcode-cn.com/problems/delete-operation-for-two-strings/)

- [712. 两个字符串的最小ASCII删除和](https://leetcode-cn.com/problems/minimum-ascii-delete-sum-for-two-strings/)

- [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

## 最小路径和类

- [64. 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)
- [174. 地下城游戏](https://leetcode-cn.com/problems/dungeon-game/)，从后往前推算。

## **股票交易问题**

状态：*`dps[i][j][k]`*表示*`i`*天内*`j`*次交易处于`k`(持有或不持有股票)状态时的最大利润。

变形：j可以为1次、无限次、k次；k可以添加格外的冷冻期等状态；可增加手续费

边界值：

dp[-1][k][0] = dp[i][0][0] = 0

dp[-1][k][1] = dp[i][0][1] = -infinity

状态转移方程，买的时候算交易一次：

不持有：$dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])$

持有时：$dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])$

情景：

- leetcode 121. 买卖股票的最佳时机 最多可以交易1次
- [剑指 Offer 63. 股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)，最多交易1次
- leetcode 122. 买卖股票的最佳时机 II 最多可以交易无限次
- leetcode 123. 买卖股票的最佳时机 III 最多可以交易2次
- leetcode 188. 买卖股票的最佳时机 IV 最多可以交易k次
- leetcode 309. 最佳买卖股票时机含冷冻期 最多可以交易无限次
- leetcode 714. 买卖股票的最佳时机含手续费 最多可以交易无限次

## 背包问题

状态：*`dp[i][j]`*表示前*`i`*个物品满足目标*`j`*的最优解。

[背包问题](https://www.notion.so/2f7bdfa95c224c63b865047a608cff9b)

# 参考

- [五大常用算法之二：动态规划算法](https://www.cnblogs.com/steven_oyj/archive/2010/05/22/1741374.html)
- [动态规划解题核心框架](https://labuladong.gitbook.io/algo/mu-lu-ye-2/mu-lu-ye/dong-tai-gui-hua-xiang-jie-jin-jie)