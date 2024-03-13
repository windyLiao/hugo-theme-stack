+++
author = "H.Liao"
title = "Union-find Set"
date = "2024-03-13"
description = "union and find?"
categories = [
    "acm",
    "graph theory",
]
image = "1.png"

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

# 图论一：并查集

并查集是一种`树型`的数据结构，用于处理一些不交集的合并和查询问题。以下有两种作用于此数据结构的操作：

* Find:确定两个元素是否属于同一子集。
* Union:将两个子集合并成同一个集合。

并查集三部曲：

* 初始化集合
* 返回x所属集合的root
* 合并x,y所在集合

> 并查集只回答两个节点是不是在一个连通分量中，即连通性问题，不回答路径问题。
>
> 用于处理动态连通性问题，最典型应用是求解最小生成树。
>
> 问题具有传递性可以考虑用并查集。

模板：

```c++
vector<int> uf; // 并查集
int Find(int x){
	//在查找时，更新集合中的关系
	return x == uf[x] ? x : uf[x] = Find(uf[x]);
}
void Union(int x,int y){
	int i = Find(x),j = Find(y);
	if(i == j) return ;
	uf[j] = uf[i];
}
```

两种优化方法：`路径压缩`、`按秩合并`。



## [547. 省份数量](https://leetcode.cn/problems/number-of-provinces/)

典型的并查集先来一道热热身。

```c++
class Solution {
    int Find(vector<int>&uf,int x){
        if (x!=uf[x]){
            uf[x]= Find(uf,uf[x]);
        }
        return uf[x];
    }
    void Union(vector<int>&uf,int x,int y){
        int X= Find(uf,x);
        int Y=Find(uf,y);
        uf[X]=Y;
    }
public:
    int findCircleNum(vector<vector<int>>& isConnected) {
        vector<int>uf(isConnected.size(),0);
        for(int i=0;i<uf.size();i++){
            uf[i]=i;
        }
        for(int i=0;i<uf.size();i++){
            for(int j=0;j<uf.size();j++){
                if(i==j)continue;
                if(isConnected[i][j]&& Find(uf,i)!= Find(uf,j)){
                    Union(uf,i,j);
                }
            }
        }
        set<int>total;
        for(int i=0;i<isConnected.size();i++){
            total.insert(Find(uf,i));
        }
        return total.size();
    }
};
```

> 最后搜索几个省的时候可以直接查询是否`uf[i]=i`.即可，不用Find之后放到集合里，减小时间复杂度。



## [684. 冗余连接](https://leetcode.cn/problems/redundant-connection/)

常规方法即可。

```c++
class Solution {
    int Find(vector<int>&uf,int x){
        if (x!=uf[x]){
            uf[x]= Find(uf,uf[x]);
        }
        return uf[x];
    }
    void Union(vector<int>&uf,int x,int y){
        int X= Find(uf,x);
        int Y=Find(uf,y);
        uf[X]=Y;
    }
public:
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        vector<int>uf(edges.size()+1,0);
        for(int i=0;i<uf.size();i++){
            uf[i]=i;
        }
        vector<pair<int,int>>record;
        for(auto & edge : edges){
            if(uf[Find(uf,edge[0])!= Find(uf,edge[1])]){
                Union(uf,edge[0],edge[1]);
            }else{
                record.emplace_back(edge[0],edge[1]);
            }
        }
        vector<int>ans(2,0);
        ans[0]=record.back().first;
        ans[1]=record.back().second;
        return ans;
    }
};
```

最后可以在循环里面优化，不用多开数组和make_pair。

类似于以下这样：

```c++
            if (p_x != p_y) pa[p_x] = p_y;
            else {
                x = edges[i][0];
                y = edges[i][1];
            }
        }
        return {x, y};
```

减少空间时间复杂度。

## [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

此题有两种方法解，其一：动态规划。

重点在于理解hash映射值的意义，即但该数字为某区间端点时记录该区间长度。

```c++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        map<int,int>hash;
        int Max=0;
        for(int i:nums){
            if(hash[i])
                continue;
            int left=hash[i-1];
            int right=hash[i+1];
            int length=left+right+1;
            hash[i]=1;
            Max=max(Max,length);
            hash[i-left]=length;
            hash[i+right]=length;
        }
        return Max;
    }
}
```

其二：并查集，将$i-1,i,i+1$合并即可，逻辑很简单，不做介绍。

## [399. 除法求值](https://leetcode.cn/problems/evaluate-division/)

这题有意思，在简单并查集的基础上增加了需要更新的参数，当然这题的重点在于如何写`Find`函数更新新参数，以及一些细节。

```c++
#include <bits/stdc++.h>
using namespace std;
#include <vector>

class Union{
private:
    vector<int>parent;
    vector<double>weight;

public://成员初始化列表
    Union(int num):parent(num+1,0),weight(num+1,1){
        for(int i=0;i<parent.size();i++)parent[i]=i;
    }

    int Find(int k){
        if(k!= parent[k]){
            int origin=parent[k];//记录原来的父节点
            parent[k]= Find(parent[k]);
            weight[k]*=weight[origin];//全部用根节点的倍数表示，同一化
        }
        return parent[k];
    }

    void couple_union(int x,int y,double weigh){
        int X= Find(x),Y= Find(y);
        if(X==Y)return;
        parent[X]=Y;
        weight[X]=weigh*weight[y]/weight[x];//记住Find之后一棵树最多两层，路径压缩过
    }

    double isconnected(int x,int y){
        if(Find(x)!= Find(y))
            return -1;
        return weight[x]/weight[y];//返回结果
    }
};

class Solution {
public:
    vector<double> calcEquation(vector<vector<string>>& equations, vector<double>& values, vector<vector<string>>& queries) {
        Union myUnion(equations.size()*2);
        int id=1,i=0;
        map<string ,int>hash;//把字符映射到数字，方便底层实现
        vector<double>ans;
        for(auto vec:equations){
            if(!hash[vec[0]])hash[vec[0]]=id++;
            if(!hash[vec[1]])hash[vec[1]]=id++;
            myUnion.couple_union(hash[vec[0]],hash[vec[1]],values[i++]);//合并
        }
        for(auto vec:queries){//查询操作
            cout<<vec[0]<<" "<<vec[1]<<endl;
            int x=hash[vec[0]];
            int y=hash[vec[1]];
            if(!(x&&y)){
                ans.push_back(-1.0);
                continue;
            }
            ans.push_back(myUnion.isconnected(x,y)?myUnion.isconnected(x,y):-1.0);
        }
        return ans;
    }
};

int main(){
    vector<vector<string>> test{{"a","b"},{"b","c"}};
    vector<double>test_value{2.0,3.0};
    vector<vector<string>>test_queries{{"a","c"},{"b","a"},{"a","e"},{"a","a"},{"x","x"}};
    Solution().calcEquation(test,test_value,test_queries);
}

```

代码部分使用了`成员初始化列表`，做个了解：

成员初始化列表（member initialization list）是在C++中用于初始化类的成员变量的一种特殊语法。它位于构造函数的参数列表之后，使用冒号`:`开头，后面跟着初始化语句。成员初始化列表可以在构造函数体执行之前初始化成员变量，有以下几个重要的特点：

1. **初始化成员变量**：成员初始化列表用于初始化类的成员变量，可以在构造函数体内对成员变量进行初始化，避免了在构造函数体内使用赋值语句进行初始化。

2. **效率**：使用成员初始化列表初始化成员变量可以提高代码的执行效率。在构造函数体内对成员变量进行赋值时，实际上是先调用默认构造函数初始化成员变量，然后再调用赋值操作符进行赋值，而成员初始化列表直接初始化成员变量，避免了这种多余的操作。

3. **初始化顺序**：成员初始化列表中的初始化顺序与成员变量在类中声明的顺序有关，与它们在构造函数初始化列表中出现的顺序无关。成员初始化列表中的初始化顺序取决于成员变量在类中声明的顺序。

4. **const和引用成员变量**：对于`const`和引用类型的成员变量，必须使用成员初始化列表进行初始化，因为它们一旦声明就必须被初始化，而且不能被重新赋值。

下面是一个示例代码，演示了如何使用成员初始化列表初始化类的成员变量：

```cpp
#include <iostream>

class MyClass {
private:
    int num;
    double value;

public:
    MyClass(int n, double v) : num(n), value(v) {
        // 构造函数体
    }
};

int main() {
    MyClass obj(10, 3.14);
    return 0;
}
```

在上面的示例中，`MyClass`类的构造函数使用成员初始化列表来初始化`num`和`value`成员变量。这种方式可以提高代码的执行效率，并确保成员变量在构造函数体内被正确初始化。

## *[803. 打砖块](https://leetcode.cn/problems/bricks-falling-when-hit/)

崩溃，看了题解自己又写了一遍，测试好几次都有问题，漏掉好多细节。题解的思路很好，感觉我这笨脑子就想不到。敲砖块的逆过程就是合并，查看多少砖头被合并到顶层，即可算出敲掉该砖块会掉落多少砖头，当然要从最后一个敲掉的开始计算。上代码。

```c++
vector<vector<int>>dir{{-1,0},{1,0},{0,-1},{0,1}};
class Union{
private:
    int row,col,size;
    vector<int>uf;
    vector<int>scale;
public:
    Union(int r,int c):row(r),col(c),size(r*c),uf(size+1,0),scale(size+1,1){
        for(int i=0;i<uf.size();i++){
            uf[i]=i;
        }
    }
    int Find(int x){
        if(x!= uf[x]){
            uf[x]= Find(uf[x]);
        }
        return uf[x];
    }
    void Union_couple(int x1,int x2,int y1,int y2){
        int x= get_index(x1,x2);
        int y= get_index(y1,y2);
        int X=Find(x),Y=Find(y);
        if(X==Y)return;
        uf[X]=Y;
        scale[Y]+= scale[X]; //维护节点个数数组
    }

    int get_size(int x){
        return scale[Find(x)];//细节size的根会改变
    }

    int get_index(int x,int y){
        return x*col+y;
    }

    bool in(int x,int y){
        return x>=0&&y>=0&&x<row&&y<col;
    }
};

class Solution {
public:
    vector<int> hitBricks(vector<vector<int>>& grid, vector<vector<int>>& hits) {
        vector<vector<int>>copy=grid;//防止敲掉的是本身不存在的砖，验证使用
        vector<int>ans(hits.size(),0);
        int r=grid.size(),c=grid[0].size();
        Union myUnion(r,c);
        int size=r*c;
        for(auto i:hits)
            grid[i[0]][i[1]]=0;//敲掉的全归0
        for(int i=0;i<grid[0].size();i++){
            if(grid[0][i]==1)
                myUnion.Union_couple(0,i,0,size);//特殊处理顶层，根下标设为size
        }
        for(int i=1;i<grid.size();i++){
            for(int j=0;j<grid[0].size();j++){
                if(grid[i][j]){ //合并现有砖头为连通，根不定无所谓
                    if(grid[i-1][j]) //遍历方向是右下，实际合并方向左上即可
                        myUnion.Union_couple(i,j,i-1,j);
                    if(j-1>=0&&grid[i][j-1])
                        myUnion.Union_couple(i,j,i,j-1);
                }
            }
        }
        for(int i=hits.size()-1;i>=0;i--){
            int pre=myUnion.get_size(size);
            int x=hits[i][0];
            int y=hits[i][1];
            if(!copy[x][y])//排除空砖
                continue;
            if(x==0)
                myUnion.Union_couple(0,y,0,size);//特殊处理顶层砖
            for(auto go:dir){
                if(myUnion.in(x+go[0],y+go[1])&&grid[x+go[0]][y+go[1]])//上下左右四个方向遍历
                    myUnion.Union_couple(x,y,x+go[0],y+go[1]);
            }
            int cur=myUnion.get_size(size);
            int num=max(0,cur-pre-1);//返回差值即为所求
            grid[x][y]=1;
            ans[i]=num;
        }
        return ans;
    }
};

```

并查集刷到这基本结构清楚了，剩下的靠智商了说实话，难题的要求就是清奇的思路，维护额外所需数组，细节多考虑。。。

