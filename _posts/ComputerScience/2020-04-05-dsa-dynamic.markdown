---
layout: post
title:  "【DSA】数据结构与算法 总结 08 动态规划"
crawlertitle: "动态规划"
summary: "Data Structures and Algorithm —— Dynamic Programming"
date:   2020-04-05 10:00:00 +0800
categories: posts
tags: 'CS'
author: xusc
bg: "CS.jpg"
---

动态规划用来解决多阶段决策过程的最优化问题。动态规划一般可分为线性动规，区域动规，树形动规，背包动规四类。

求解方法：
1. **划分阶段**：按照问题的时间或空间特征，把问题分为若干个阶段。在划分阶段时，注意划分后的阶段一定要是有序的或者是可排序的，否则问题就无法求解。
2. **确定状态和状态变量**：将问题发展到各个阶段时所处于的各种客观情况用不同的状态表示出来。状态的选择要满足无后效性。
3. **确定决策并写出状态转移方程**：因为决策和状态转移有着天然的联系，状态转移就是根据上一阶段的状态和决策来导出本阶段的状态。所以如果确定了决策，状态转移方程也就可写出。但事实上常常是反过来做，根据相邻两段各状态之间的关系来确定决策。
4. **寻找边界条件**：给出的状态转移方程是一个递推式，需要一个递推的终止条件或边界条件。

### 子序列问题
[最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)
```java
Arrays.fill(dp, 1);
for (int i = 1; i < n; i++)
    for (int j = 0; j < i; j++)
        if (nums[j] < nums[i])
            dp[i] = Math.max(dp[j] + 1, dp[i]);
```

[最长递增子序列的个数](https://leetcode-cn.com/problems/number-of-longest-increasing-subsequence/)
```java
Arrays.fill(dp, 1);
Arrays.fill(combination, 1);
for (int i = 1; i < dp.length; i++) {
    for (int j = 0; j < i; j++) {
        if (nums[j] < nums[i]) {
            if (dp[j] + 1 > dp[i]) {        // 如果 +1 长于当前 LIS 则组合数不变
                dp[i] = dp[j] + 1;
                combination[i] = combination[j];
            } else if (dp[j] + 1 == dp[i])  // 如果 +1 等于当前 LIS 则说明找到了新组合
                combination[i] += combination[j];
        }
    }
    max = Math.max(max, dp[i]);
}
```

[最长连续递增子序列](https://leetcode-cn.com/problems/longest-continuous-increasing-subsequence/)
```java
Arrays.fill(dp, 1);
for(int i = 1; i < n; i++){
    if(nums[i] > nums[i - 1])
        dp[i] = dp[i - 1] + 1;
}
```

[最大子序列和](https://leetcode-cn.com/problems/maximum-subarray/)
```java
for(int i = 1; i < n; i++)
    dp[i] = Math.max(dp[i - 1], 0) + nums[i];
    max = Math.max(dp[i], max);
}
```

[最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)
```java
for(int i = 1; i < n1 + 1; i++)
    for(int j = 1; j < n2 + 1; j++)
        if(s1[i - 1] == s2[j - 1])
            dp[i][j] = dp[i - 1][j - 1] + 1;
        else
            dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
```

最长公共子串
```java
for(int i = 1; i < n1 + 1; i++)
    for(int j = 1; j < n2 + 1; j++)
        if(s1[i - 1] == s2[j - 1])
            dp[i][j] = dp[i - 1][j - 1] + 1;
        else
            dp[i][j] = 0;
```


### 背包动规
**0/1 背包**：有 `n` 件物品和一个容量为 `m` 的背包。第 `i` 件物品的体积是 `w[i]`，价值是 `v[i]`。求解将哪些物品装入背包可使这些物品的体积总和不超过背包容量，且价值总和最大。
+ 阶段：物品的件数
+ 状态：背包剩余的容量
+ 状态转移方程：***`f[i][j] = max{f[i - 1][j], f[i - 1][j - w[i]] + v[i]}`***
  + 将前 i 个物品放入容量为 j 的背包中的最大价值
  + 如果不放入 i ，最大价值就和 i 无关，即 `f[i - 1][j]`；如果放入 i，就是 `f[i - 1][j - w[i]] + v[i]`
+ 空间优化，采用一维数组：***`f[j] = max{f[j], f[j - w[i]] + v[i]}`***

```java
for(int i = 1; i <= n; i++)
    for(int j = m; j > 0; j--)
        if(j >= w[i])
            f[j] = max(f[j], f[j - w[i]] + v[i]);
```

**完全背包问题**：有 `n` 种物品和一个容量为 `m` 的背包，每种物品都有无限件可用。第 `i` 种物品的体积是 `w[i]`，价值是 `v[i]`。求解将哪些物品装入背包可使这些物品的体积总和不超过背包容量，且价值总和最大。
+ 状态转移方程：***`f[i][j] = max{f[i - 1][j], f[i - 1][j - k * w[i]] + k * v[i]}, 1 <= k <= m / w[i]`***
+ 优化
  + 变形一：让 k 可取 0.
    + `f[i][j] = max{f[i - 1][j - k * w[i]] + k * v[i]}, 0 <= k <= m / w[i]` ... A
  + 变形二：比较*不放第 i 种物品*和*放第 i 种物品 k(k >= 1) 件中结果最大的 k*。
    + `f[i][j] = max{f[i - 1][j], max{f[i - 1][j - k * w[i]] + k * v[i]}}, 1 <= k <= m / w[i]`
  + 变形三：在第 i 件物品已经放入 1 件的状态下再考虑放入 k(k >= 0) 件这种物品的结果是否更大。
    + `f[i][j] = max(f[i - 1][j], max{f[i - 1][(j - w[i]) - k * w[i]] + k * v[i]} + v[i]), 0 <= k <= m / w[i]`
  + 变形四：根据 A 式，`f[i][j - w[i]] = max{f[i - 1][(j - w[i]) - k * w[i]]`
    + ***`f[i][j] = max{f[i - 1][j], f[i][j - w[i]] + v[i]}`***
+ 空间优化，采用一维数组：***`f[j] = max(f[j], f[j - w[i]] + v[i])`***
  + 与 0/1 背包问题不同的是，第二层循环是正序

```java
for(int i = 1; i <= n; i++)
    for(int j = 0; j <= m; j++)
        if(j >= w[i])
            f[j] = max(f[j], f[j - w[i]] + v[i]);
```

**多重背包问题**：有 `n` 种物品和一个容量为 `m` 的背包，第 `i` 种物品，有 `x[i]` 件可用体积是 `w[i]`，价值是 `v[i]`。求解将哪些物品装入背包可使这些物品的体积总和不超过背包容量，且价值总和最大。
+ 状态转移方程：***`f[i][j] = max{f[i - 1][j], f[i - 1][j - k * w[i]] + k * v[i]}, 1 <= k <= x[i]`***

```java
for(int i = 1; i <= n; i++)
	for(int j = m; j > 0; j--)
		for(int k = 0; k <= x[i]; k++)
			if(j >= k * w[i])
				f[j] = max(f[j], f[j - k * w[i]] + k * val[i]);
```