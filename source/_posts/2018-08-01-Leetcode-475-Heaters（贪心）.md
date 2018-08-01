---
title: Leetcode 475. Heaters（贪心）
urlname: leetcode-475-heaters
toc: true
date: 2018-08-01 12:26:27
updated: 2018-08-01 18:43:00
mathjax: true
tags: [Leetcode]
---


题目来源：[https://leetcode.com/problems/heaters/description/](https://leetcode.com/problems/heaters/description/)

标记难度：Easy

提交次数：1/1

代码效率：18.81%

## 题意

x轴上有若干个暖气和若干个房子，每个暖气能够覆盖到半径为x的圆内的房子，问至少取x为多少，才能覆盖所有房子。（所有暖气的半径是一样的）

## 分析

如果需要排序，则时间复杂度下限应该是$O(n \log{n})$。排序之后，很显然可以立刻利用二分查找（`lower_bound`或`upper_bound`）来寻找距离每所房子最近的暖气，并确定所需的最小半径。如果房子和暖气开始时都是排好序的，则很显然有$O(n)$的解法，因为此时随着房子坐标的递增，它在暖气中的`lower_bound`必然也是单调递增的，因此扫描一遍所有暖气就够了，具体解法可参见[C++ O(N) speed O(1) space](https://leetcode.com/problems/heaters/discuss/95927/C++-O%28N%29-speed-O%281%29-space)。

<!--
只好把()转义成%28和%29了
-->

以及我每次都记不住[upper_bound](https://zh.cppreference.com/w/cpp/algorithm/upper_bound)的用法。

* 调用方法：`upper_bound(vector.begin(), vector.end(), val)`
* 返回值：`iterator`（虽然这么说不算很准确）
* 返回值使用方法：
  * `*iterator`：`lower_bound`的数值
  * `iterator - vector.begin()`：`lower_bound`对应的数在数组中的下标
  * `iterator != vector.end()`：是否找到了`lower_bound`

## 代码

```
class Solution {
public:
    int findRadius(vector<int>& houses, vector<int>& heaters) {
        // 既然标签是Easy，那我觉得大概是个贪心问题，至多是动态规划问题
        // （当然这个思考方式是不好的）
        // 又看错题了，原来所有的暖气的半径要求是一样的。
        // 我感觉有O(n)的解法，但是显然直接O(n * log(n))比较方便。

        if (houses.size() <= 0 || heaters.size() <= 0)
            return 0;

        int n = houses.size(), m = heaters.size();
        sort(houses.begin(), houses.end());
        sort(heaters.begin(), heaters.end());

        // 我决定为每一个房子找到离它最近的暖气。
        int ans = 0;
        for (int i = 0; i < n; i++) {
            auto ub = upper_bound(heaters.begin(), heaters.end(), houses[i]);
            int dist;
            // 此house的坐标>=所有heater的坐标
            if (ub == heaters.end())
                dist = houses[i] - heaters[m-1];
            else {
                int j = ub - heaters.begin();
                dist = heaters[j] - houses[i];
                if (j > 0)
                    dist = min(dist, houses[i] - heaters[j-1]);
            }
            ans = max(ans, dist);
        }

        return ans;
    }
};
```
