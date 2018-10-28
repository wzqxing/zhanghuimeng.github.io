---
title: Leetcode 932. Beautiful Array（数学）
urlname: leetcode-932-beautiful-array
toc: true
date: 2018-10-28 15:58:38
updated: 2018-10-28 16:38:38
tags: [Leetcode, Leetcode Contest, alg:Math]
---

题目来源：[https://leetcode.com/problems/beautiful-array/description/](https://leetcode.com/problems/beautiful-array/description/)

标记难度：Medium

提交次数：2/3

代码效率：

* 直接递归版本：8ms
* 带cache递归版本：12ms

## 题意

对于正整数`N`，称1到`N`的全排列`A`为beautiful当且仅当对于任意`i < j`，不存在满足`i < k < j`且`A[k] * 2 = A[i] + A[j]`的`k`。给定`N`，请输出任意beautiful的`A`。

## 分析

一道很有趣的题。（这次比赛最难的题就是medium了啊）最开始我并没有什么思路。于是我暴力了一下：

`N=4`时的beautiful排列共有10个：

```
1 3 2 4
1 3 4 2
2 1 4 3
2 4 1 3
2 4 3 1
3 1 2 4
3 1 4 2
3 4 1 2
4 2 1 3
4 2 3 1
```

`N=5`时共有20个：

```
1 5 3 2 4
1 5 3 4 2
2 1 4 5 3
2 4 1 5 3
2 4 3 1 5
2 4 3 5 1
2 4 5 1 3
...
```

`N=6`时共有48个：

```
1 5 3 2 6 4
1 5 3 4 2 6
1 5 3 4 6 2
1 5 3 6 2 4
1 5 6 3 2 4
...
```

`N=7`时共有104个：

```
1 5 3 2 7 6 4
1 5 3 7 2 6 4
1 5 3 7 4 2 6
1 5 3 7 4 6 2
1 5 3 7 6 2 4
1 5 7 3 2 6 4
1 5 7 3 4 2 6
1 5 7 3 4 6 2
1 5 7 3 6 2 4
1 5 7 6 3 2 4
...
```

我猜beautiful排列的数量肯定是随`N`递增的，但和`N!`比起来大概是稀疏的，所以随机生成一个排列然后再判断它是否具有所需的性质多半不太可取。然后通过观察，我发现很多beautiful排列都是把奇数排在前面，偶数排在后面的！于是就写了一个交上去了。我心想，这样做的道理是，不需要处理偶数和奇数中间的`k`；两个奇数相加除2如果生成偶数，则偶数肯定不在这些奇数里面……

于是显然就WA了，没有注意到两个奇数相加除2也可能会生成奇数这一点。

但这就使得我注意到了一种可能存在的递归结构：以`N=7`为例，先把所有数分成奇数`1 3 5 7`和偶数`2 4 6`，然后把`1 3 5 7`转换成`1 2 3 4`，`2 4 6`转换成`1 2 3`，分别递归执行，得到`1 3 2 4`和`1 3 2`之后，再转换回`1 5 3 7`和`2 6 4`，把它们拼起来，就可以得到最终结果`1 5 3 7 2 6 4`。关于这一点的更正式的证明是，beautiful数组（经过转换之后不一定是排列了）满足以下性质[^lee215]：

* 若`A`是beautiful数组，则`A + x`（对数组的每个元素加`x`）仍是beautiful数组
* 若`A`是beautiful数组，则`A * x`（对数组的每个元素乘`x`）仍是beautiful数组
* 若`A`是beautiful数组，则删除`A`中的一些元素，`A`仍是beautiful数组

以及lee215的题解[^lee215]质量过于好了，真的，十分推荐阅读，除此之外，还有[one-liner版本](https://leetcode.com/problems/beautiful-array/discuss/186680/Python-1-line-solutions)（仿佛看到了StefanPochmann……）

[^lee215]: [lee215's Solution for Leetcode 932: \[C++/Java/Python\] Odd + Even Pattern, O(N)](https://leetcode.com/problems/beautiful-array/discuss/186679/C++JavaPython-Odd-+-Even-Pattern-O%28N%29)

## 代码

### 直接递归

应该把这个函数直接写到`beautifulArray`函数里去的，而且算法也不是DFS……

```cpp
class Solution {
private:
    vector<int> dfs(int N) {
        if (N == 1) return {1};
        // 奇数：(x + 1) / 2
        // 偶数：x / 2
        vector<int> left = dfs((N + 1) / 2);
        vector<int> right = dfs(N / 2);
        vector<int> sum;
        for (int x: left)
            sum.push_back(x * 2 - 1);
        for (int x: right)
            sum.push_back(x * 2);
        return sum;
    }

public:
    vector<int> beautifulArray(int N) {
        return dfs(N);
    }
};
```

### 带cache的递归

```cpp
class Solution {
private:
    unordered_map<int, vector<int>> cache;

    vector<int> backtrack(int N) {
        if (cache.find(N) != cache.end()) return cache[N];
        // 奇数：(x + 1) / 2
        // 偶数：x / 2
        vector<int> left = dfs((N + 1) / 2);
        vector<int> right = dfs(N / 2);
        vector<int> sum;
        for (int x: left)
            sum.push_back(x * 2 - 1);
        for (int x: right)
            sum.push_back(x * 2);
        cache[N] = sum;
        return sum;
    }

public:
    vector<int> beautifulArray(int N) {
        cache[1] = {1};
        return backtrack(N);
    }
};
```
