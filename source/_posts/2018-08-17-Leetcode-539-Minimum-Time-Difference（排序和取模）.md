---
title: Leetcode 539. Minimum Time Difference（排序和取模）
urlname: leetcode-539-minimum-time-difference
toc: true
date: 2018-08-17 10:38:34
updated: 2018-08-17 15:01:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/minimum-time-difference/description/](https://leetcode.com/problems/minimum-time-difference/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* 普通排序：25.78%
* 优化和桶排序：89.80%

## 题意

给定若干个时间点（格式为`HH:mm`），求其中最小的时间间隔。

## 分析

总的来说是道非常水的题，直接把小时全都化成分钟，然后排序求间隔就可以了。唯一需要注意的是排序完之后开头和末尾的时间需要特殊处理。（就像12月和1月也是相连的那样。）

题解区里有很多很妙的想法。比如，[有人声称](https://leetcode.com/problems/minimum-time-difference/discuss/100694/7-liner-%2522O%281%29%2522-solution%3A-only-60*24-possible-different-time-points!)，最多只有`60*24`个时间点，如果输入数据超过这一规模，则可以直接返回`0`，因为根据鸽巢原理，必然有重复的时间。因此，我们最多只需处理`60*24`规模的数据，所以这个算法是`O(1)`的！我觉得，这个算法由于格式限制，是无法scale的，所以说是`O(1)`倒也没有什么太大的问题，虽然这么说没什么意义。

因为数据规模的原因，显然可以用桶排序进行优化。

## 代码

### 普通排序

```cpp
class Solution {
public:
    int findMinDifference(vector<string>& timePoints) {
        // 可以转换成int，然后计算模1440情况下的最小差值
        vector<int> intTimes;
        for (string pnt: timePoints) {
            int minutes = stoi(pnt.substr(0, 2)) * 60 + stoi(pnt.substr(3, 2));
            intTimes.push_back(minutes);
        }
        sort(intTimes.begin(), intTimes.end());
        int minDif = (intTimes.front() - intTimes.back() + 1440) % 1440;
        for (int i = 1; i < intTimes.size(); i++)
            minDif = min(minDif, intTimes[i] - intTimes[i - 1]);
        return minDif;
    }
};
```

### 优化和桶排序

快了大约8ms。

```cpp
class Solution {
public:
    int findMinDifference(vector<string>& timePoints) {
        if (timePoints.size() > 1440)
            return 0;
        // 可以转换成int，然后计算模1440情况下的最小差值
        int bucket[1440];
        memset(bucket, 0, sizeof(bucket));
        for (string pnt: timePoints) {
            int minutes = stoi(pnt.substr(0, 2)) * 60 + stoi(pnt.substr(3, 2));
            bucket[minutes]++;
            if (bucket[minutes] > 1)
                return 0;
        }
        int first = -1, last = -1, minDif = 1440;
        for (int i = 0; i < 1440; i++)
            if (bucket[i] != 0) {
                if (first == -1)
                    first = i;
                if (last != -1)
                    minDif = min(minDif, i - last);
                last = i;
            }
        minDif = min(minDif, (first - last + 1440) % 1440);
        return minDif;
    }
};
```
