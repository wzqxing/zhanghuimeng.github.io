---
title: Leetcode 975. Odd Even Jump（栈）
urlname: leetcode-975-odd-even-jump
toc: true
date: 2019-01-16 02:20:44
updated: 2019-01-17 15:46:00
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming, alg:Stack, alg:Monotonic Stack]
---

题目来源：[https://leetcode.com/problems/odd-even-jump/description/](https://leetcode.com/problems/odd-even-jump/description/)

标记难度：Hard

提交次数：2/2

代码效率：

* 单调栈：100.00%（72ms）
* 平衡树：60.56%（100ms）

## 题意

>We need to jump higher and lower alternately to the end. [^lee215]

[^lee215]: [lee215's solution for 975 - \[Java/C++/Python\] DP idea, Using TreeMap or Stack](https://leetcode.com/problems/odd-even-jump/discuss/217981/JavaC++Python-DP-idea-Using-TreeMap-or-Stack)

这句题目描述太精妙了，以至于我深感自己理解能力和语言描述能力的匮乏。

简单来说就是，从数组的某个index出发，先跳到它后面的比它大的元素中最小的元素（如果有相同元素则选择最靠左的一个），再跳到它后面的比它小的元素中最大的元素（相同时仍然选择最靠左的一个），交替采取这两种跳法，直到跳不动了为止。问从多少个index出发可以最终跳到最后一个元素。

## 分析

### 法一：单调栈

我在做题之前得到了一点提示（这道题应该用栈来做），遂在一通魔调之后搞出了一个能过的单调栈解法。说起来，我还是看了这次的题解才知道这种方法的名字应该叫单调栈（monotonic stack）。于是我决定把之前用这个算法的文章的标签修改一下……

和之前的那些单调栈相比，这次的做法没什么大的差异，但解决平局的方法有所不同。以index为第二关键字对数组排序之后，找右侧更大的数的时候，需要倒序遍历数组，单调栈正好可以处理平局的问题；但找右侧更小的数的时候，正序遍历数组，对于大小相同的数，index较小的先出现，较大的后出现，之后的数就找不到index较小的数了。

所以找右侧更小的数的时候，需要把大小相同的数按index倒序排序。

单调栈中index的大小和左右侧有关，排序的顺序和大小有关。

### 法二：平衡树

但是实际上我已经做单调栈的题太多以至于过拟合了。[之前](/post/leetcode-739-daily-temperatures)我还会想到平衡树的解法的……现在已经只会单调栈啦！！！[^lee215]

结果在这个解法上还是遇到了问题。第一个问题就是不要用`std::upper_bound`，要用`map.upper_bound`（针对内部容器实现的，和针对`map`实现的区别）。第二个问题是，题目要求是带等号的，而`upper_bound`找不到等号，所以要修改一下得到的结果。

## 代码

### 单调栈

```cpp
class Solution {
public:
    int oddEvenJumps(vector<int>& A) {
        int n = A.size();
        int odd[n], even[n];
        
        stack<pair<int, int>> s;
        vector<pair<int, int>> indices;
        for (int i = 0; i < n; i++)
            indices.emplace_back(A[i], -i);
        sort(indices.begin(), indices.end());
        for (int i = 0; i < n; i++)
            indices[i].second = -indices[i].second;
        for (int i = 0; i < n; i++) {
            while (!s.empty() && indices[i].second > s.top().second) {
                s.pop();
            }
            if (s.empty())
                even[indices[i].second] = -1;
            else
                even[indices[i].second] = s.top().second;
            s.push(indices[i]);
        }
        
        s = stack<pair<int, int>>();
        indices.clear();
        for (int i = 0; i < n; i++)
            indices.emplace_back(A[i], i);
        sort(indices.begin(), indices.end());
        for (int i = n - 1; i >= 0; i--) {
            while (!s.empty() && indices[i].second > s.top().second)
                s.pop();
            if (s.empty())
                odd[indices[i].second] = -1;
            else
                odd[indices[i].second] = s.top().second;
            s.push(indices[i]);
        }
        
        int oddjump[n], evenjump[n];
        int ans = 0;
        for (int i = n - 1; i >= 0; i--) {
            if (odd[i] != -1) oddjump[i] = evenjump[odd[i]];
            else oddjump[i] = i;
            if (even[i] != -1) evenjump[i] = oddjump[even[i]];
            else evenjump[i] = i;
            if (oddjump[i] == n - 1) ans++;
        }
        return ans;
    }
};
```

### 平衡树

```cpp
class Solution {
public:
    int oddEvenJumps(vector<int>& A) {
        int n = A.size();
        int odd[n], even[n];
        even[n-1] = odd[n-1] = n - 1;
        
        int ans = 1;
        map<int, int> treeSet;
        treeSet[A[n-1]] = n - 1;
        for (int i = n - 2; i >= 0; i--) {
            auto si = treeSet.upper_bound(A[i]);
            auto bi = treeSet.lower_bound(A[i]);
            if (si != treeSet.begin())
                even[i] = odd[(--si)->second];
            else
                even[i] = -1;
            if (bi != treeSet.end())
                odd[i] = even[bi->second];
            else
                odd[i] = -1;
            treeSet[A[i]] = i;
            if (odd[i] == n - 1) ans++;
        }
        
        return ans;
    }
};
```