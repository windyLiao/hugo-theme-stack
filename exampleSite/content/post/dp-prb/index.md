+++
author = "HansonLiao"
title = "动态规划：01背包"
date = "2024-03-06"
description = "let's begin with dp problems"
categories = [
    "acm",
    "dp","01-bag"
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

# 背包问题一：0-1背包

## 1.状态转移方程

### 1.1 概念

01背包指每种物品只能选择一次，计算出最大价值.

### 1.2 基础状态转移方程

$f[i][j] = max\{f[i-1][j], f[i-1][j-w[i]] + v[i]\}$

子问题是**将前i件物品放入容量为v的背包中**，只考虑第i件物品放或不放，就可以把问题转化为"前i-1件物品放入容量为v的背包中",价值为$f[i-1][v]$;若放第i件物品，则理解为"前i-1件物品放入容量为v-c[i]的背包中"，此时获得的价值为$f[i-1][v-c[i]]+w[i]$.

## 2. 代码及优化

### 2.1 代码

```c++
//f[i][j]的意义是表示只看前i个物品，总体积是j的情况下，总价值最大是多少
for(int i = 1; i <= n; ++i){
    for(int j = 0; j <= C; ++j){
		if(j >= v[i])	dp[i][j]=max(dp[i-1][j],dp[i-1][j-v[i]] + w[i]); //取
		else	dp[i][j] = dp[i-1][j];		//不取第i个物品
    }
}
cout<<dp[n][C]<<endl;
```

### 2.2 空间优化

> 进行一维空间优化时注意是**逆序**的，否则会提前覆盖需要引用的提前量

```C++
//f[i]的意义为当前体积为i的情况下背包的最大价值	·
for(int i = 1;i <= n;++i)
        for(int j = C;j >= w[i]; --j)
            dp[j] = max(dp[j], dp[j-w[i]] + v[i]);
cout << dp[C] << endl;
```

### 2.3初始化

* 求解背包容量是C的**最大价值**，将f数组全初始化为0；

* 求解背包体积恰好为C的情况下其最大价值，只需将$f[0]$初始化为0，其他的初始化为-inf

## 3.Leecode相关题目

### [416. 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)

#### 常规背包解法(一维空间优化)：

```c++
#include <bits/stdc++.h>
using namespace std;
#include <vector>

class Solution  {
public:
    bool canPartition(vector<int>& nums){
        int sum=accumulate(nums.begin(),nums.end(),0);
        if(sum%2)
            return false; //奇数返回false
        int tar=sum/2; //背包容量
        vector<int> dp(tar+1,0);
        for(int i=0;i<nums.size();i++){
            for(int v=tar;v>=0;v--){
                if(v<nums[i])
                    break;
                dp[v]=max(dp[v],dp[v-nums[i]]+nums[i]);
            }
        }
        return dp[tar]==tar;
    }
};

int main(){
    vector<int> num{1,5,11,5};
    cout<<Solution().canPartition(num)<<endl;
}
```

思路：将数组分割为值相等的两个子集，可以联想到：

* 背包体积为sum/2(一个背包最大值达到sum/2，剩下一个背包则相等)
* 背包要放入的物品重量为元素的值，价值也为元素的值
* 每个元素不可重复放入

#### 大佬做法

通过01移位得到所有可能，遍历一遍数组即可(很牛的思路)：

```c++
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = 0;
        for(auto num : nums) {
            sum += num;
        }
        if(sum & 1) return false;
        sum /= 2;
        bitset<10001> bt;
        bt[0] = 1;
        for(auto num : nums) {
            bt |= bt << num; //关键步骤
        }
        return bt[sum]; //检查目标是否可达
    }
};
```

> 建立$bitset$数组，$bit[0]$代表初始为空集合，遍历数组的时候移位求或，表示将新加进来的元素加入了所有的和的可能子集中。
>
> 举例：
>
> - 假设数组中有元素 `2`，初始时 `bt` 只包含了空子集，即 `bt = {1, 0, 0, 0, ...}`。
> - 在遍历到元素 `2` 时，将 `bt` 向左位移 `2` 位，得到 `{0, 0, 1, 0, ...}`，表示可以得到和为 `2` 的子集。
> - 利用按位或运算，将新得到的子集和合并到原来的 `bt` 中，得到 `{1, 0, 1, 0, ...}`。

##### $bitset$类

`bitset` 是 C++ 标准库中的一个类，用于表示固定大小的位集合，即包含固定数量的二进制位。每个位的状态可以是 0 或 1，因此 `bitset` 可以用于存储二进制表示的信息。以下是对 `bitset` 类的一些关键特性的解释：

1. **固定大小：**
   `bitset` 是一个固定大小的容器，其大小在创建时就被确定，并且在整个生命周期中保持不变。例如，`bitset<8>` 表示包含8个二进制位的位集合。

2. **初始化和访问：**

   ```cpp
   bitset<8> bt; // 初始化一个包含8位的位集合，所有位都被设为0
   bt[2] = 1;    // 设置第3位为1
   ```

   可以使用角括号内的数字来指定 `bitset` 的大小，然后可以通过索引访问和修改每一位的状态。

3. **位运算：**
   `bitset` 支持位运算，包括按位与（`&`）、按位或（`|`）、按位异或（`^`）等。这使得 `bitset` 在处理二进制数据时非常方便。

   ```cpp
   bitset<4> bt1("1010");
   bitset<4> bt2("0110");
   bitset<4> result = bt1 & bt2; // 按位与操作
   ```

4. **输入输出操作：**
   `bitset` 可以通过流操作符进行输入和输出，方便进行转换。

   ```cpp
   bitset<4> bt;
   cin >> bt; // 从输入流读取二进制表示
   cout << bt; // 输出二进制表示到输出流
   ```

5. **位集合操作：**
   `bitset` 提供了一些成员函数用于执行位集合的常见操作，例如 `count()` 计算位集合中设置为1的位的数量。

   ```cpp
   bitset<8> bt("11001100");
   int count = bt.count(); // 计算设置为1的位的数量
   ```

6. **转换为整数：**
   `bitset` 可以通过 `to_ulong()` 和 `to_ullong()` 成员函数将其内容转换为相应的无符号整数类型。

   ```cpp
   bitset<4> bt("1010");
   unsigned long decimalValue = bt.to_ulong();
   ```

总体而言，`bitset` 提供了一种简便的方式来处理和操作二进制数据，特别适用于需要对位进行操作的场景，比如位运算和位集合的表示。



### [1049. 最后一块石头的重量 II](https://leetcode.cn/problems/last-stone-weight-ii/)

看懂题目的意思是关键，其实就是把石头分为两堆使两堆的质量差最小。和前几题套路一样，直接上代码。

常规：

```c++
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int total= accumulate(stones.begin(),stones.end(),0);
        int sum=total/2;
        vector<int>dp(sum+1,0);
        for(int i=0;i<stones.size();i++){
            for(int j=sum;j>=0;j--){
                if(j<stones[i])
                    break;
                dp[j]=max(dp[j],dp[j-stones[i]]+stones[i]);
            }
        }
        return total-dp[sum]*2;
    }
};  
```

空间换时间：

```c++
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int total= accumulate(stones.begin(),stones.end(),0);
        int sum=total/2;
        bitset<3001>bt(0);
        bt[0]=1;
        for(int i:stones)
            bt|=bt<<i;
        while(!bt[sum--]);
        return total-((sum+1)<<1);
    }
};  
```



### [494. 目标和](https://leetcode.cn/problems/target-sum/)

> 经典解法，先找出状态转移方程。定义$dp[i][j]$的意义，指在考虑第`i`个数的时候，值`凑成j`的方法总数。由于每个数可加可减，不难得出状态转移方程：
>
> $dp[i][j]=dp[i-1][j-nums[i]]+dp[i-1][j+nums[i]]$
>
> 要着重考虑的是负数对数组范围的影响，因此统一偏移`sum`。

```c++
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum=0;
        for(int i:nums)
            sum+=abs(i);
        if(abs(sum)<abs(target))
            return 0;
        vector<vector<int>> dp(nums.size()+1,vector<int>(2*sum+1,0));
        dp[0][sum]=1;
        for(int i=1;i<=nums.size();i++){
            for(int j=-sum;j<=sum;j++){
                if(j-nums[i-1]>=-sum)
                    dp[i][j+sum]+=dp[i-1][j-nums[i-1]+sum];
                if(j+nums[i-1]<=sum)
                    dp[i][j+sum]+=dp[i-1][j+nums[i-1]+sum];
            }
        }
        return dp[nums.size()][target+sum];
    }
};
```

优化：假设将这组数字划分为负值部分和非负值部分，令负值部分绝对值和为`m`，则非负和为`sum-m`,则$target=s-2*m$，即

$m=\frac{s-target}{2}$,此时问题转化为**只使用 +++ 运算符，从 `nums` 凑出 mm\*m\* 的方案数**，简化了问题的负责性，不用考虑数组越界的问题，规避额外的状态值。每个数值有「选」和「不选」两种决策，转移方程为：$f[i][j]=f[i−1][j]+f[i−1][j−nums[i−1]]$.

### [474. 一和零](https://leetcode.cn/problems/ones-and-zeroes/)

每次遇到类似背包的问题，一定先思考`背包的i和j以及dp数组`代表着什么。

从题目出发，可以设i为零的个数，j为一的个数，dp数组代表当前i,j情况下最大的子集数。这样去遍历整个string数组即可。

```c++
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        vector<vector<int>>dp(m+1,vector<int>(n+1,0));
        for(string s : strs){
            int zeronum=0,onenum=0;
            for(char c:s){
                if(c=='0')zeronum++;
                else onenum++;
            }
            for(int i=m;i>=zeronum;i--){
                for(int j=n;j>=onenum;j--){
                    dp[i][j]=max(dp[i][j],dp[i-zeronum][j-onenum]+1);
                }
            }
        }
        return dp[m][n];
    }
};
```

