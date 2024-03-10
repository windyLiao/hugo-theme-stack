+++
author = "H.Liao"
title = "动态规划：完全背包"
date = "2024-03-10"
description = "let's do further"
categories = [
    "acm",
    "dp","Complete-bag"
]

math= "true"

+++

{{< math.inline >}}
{{ if or .Page.Params.math .Site.Params.math }}
<!-- KaTeX -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.css" integrity="sha384-zB1R0rpPzHqg7Kpt0Aljp8JPLqbXI3bhnPWROx27a9N0Ll6ZP/+DiW/UqRcLbRjq" crossorigin="anonymous">

<script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.js" integrity="sha384-y23I5Q6l+B6vatafAwxRu/0oK/79VlbSz7Q9aiSZUvyWYIYsd+qj+o24G5ZU2zJz" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/contrib/auto-render.min.js" integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>

{{ end }}
{{</ math.inline >}}

# 背包问题二：完全背包


"01背包"和"完全背包"都是背包问题的经典变体，它们之间的主要区别在于物品的选取方式和约束条件。以下是它们的主要区别的总结：

1. **物品选取方式：**
   - **01背包：** 对于每个物品，只有选择拿取或不拿取两种状态，即每种物品只能选择一次。
   - **完全背包：** 每个物品可以选择拿取任意多次，或者不拿取。
2. **约束条件：**
   - **01背包：** 背包的容量是有限的，每个物品只能选择一次，即每个物品有一个固定的数量。
   - **完全背包：** 背包的容量依然是有限的，但每个物品可以选择拿取多次，数量没有限制。
3. **状态转移方程：**
   - **01背包：** 设 `dp[i][j]` 表示前i个物品，容量为j时的最优值，则状态转移方程通常为 `dp[i][j] = max(dp[i-1][j], dp[i-1][j-w[i]] + v[i])`，表示在考虑第i个物品时，选择拿取或不拿取的最优值。
   - **完全背包：** 类似地，状态转移方程为 `dp[i][j] = max(dp[i-1][j], dp[i][j-w[i]] + v[i])`，表示在考虑第i个物品时，可以选择多次拿取。

直接上题目感受一下。

## [518. 零钱兑换 II](https://leetcode.cn/problems/coin-change-ii/)

典型的可以用滚动数组解法，考虑dp数组含义为前i个零钱选择最后数值为j的方法数，则状态转移关系：

$dp[i][j]=dp[i-1][j]+dp[i-1][j-coins[i]]$.

理解为在考虑第i个零钱必须拿的方法数为$dp[i-1][j-coins[i]]$,值得注意的是顺序必须`正序`.

优化后代码：

```c++
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        vector<int>dp(amount+1,0);
        dp[0]=1;
        for(int i=0;i<coins.size();i++){
            for(int j=coins[i];j<=amount;j++){
                dp[j]+=dp[j-coins[i]];
            }
        }
        return dp[amount];
    }
};
```



## [377. 组合总和 Ⅳ](https://leetcode.cn/problems/combination-sum-iv/)

和前一题的区别是顺序不同也算一种，其实就是把里外循环换了一下，这样每个目标数值会考虑所有的元素值。

```c++
class Solution {
public:
    int combinationSum4(vector<int>& nums, int target) {
        vector<int>dp(target+1,0);
        dp[0]=1;
        for(int j=0;j<=target;j++){
            for(int i:nums)
                if(j>=i)
                    dp[j]+=dp[j-i];
        }
        return dp[target];
    }
};
```



## [322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

这题的区别是完全背包找最小值，所有注意初始化为`INT_MAX`,即可。

```c++
class Solution {
public:
    int coinChange(vector<int> &coins, int amount)
{
    vector<long long> dp(amount + 1, INT_MAX); //给dp数组每个位置赋初值为INT_MAX是为了最后判断是否能填满amount,要用long long 类型
    dp[0] = 0;  //dp[i]:换到面值i所用的最小数量
    for (int coin : coins)
    {
        for (int i = 0; i <= amount; i++)
        {
            if (coin <= i)
                dp[i] = min(dp[i], dp[i - coin] + 1);
        }
    }
    return dp[amount] == INT_MAX ? -1 : dp[amount];
}
};
```



## [279. 完全平方数](https://leetcode.cn/problems/perfect-squares/)

一样的方法，初始化注意一下，dp[0]初始化为0，其余为正无穷。

```c++
class Solution {
public:
    int numSquares(int n) {
        int l= floor(sqrt(n));
        vector<int>dp(n+1,INT_MAX);
        dp[0]=0;
        for(int i=1;i<=l;i++){
            for(int j=i*i;j<=n;j++){
                dp[j]=min(dp[j],dp[j-i*i]+1);
            }
        }
        return dp[n];
    }
};
```



## [139. 单词拆分](https://leetcode.cn/problems/word-break/)

注意双循环的顺序。

```c++
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        vector<bool>dp(s.size()+1, false);
        dp[0]= true;
        for(int j=1;j<=s.size();j++){
            for(int i=0;i<wordDict.size();i++) {
                if(wordDict[i].size()>j)
                    continue;
                if (dp[j - wordDict[i].size()] && s.substr(j - wordDict[i].size(), wordDict[i].size()) == wordDict[i]) {
                    dp[j] = true;
                }
            }
        }
        return dp[s.size()];
    }
};
```

关于二者：**遍历顺序决定背包类型(重复/不重复)，循环顺序决定有序无序**

