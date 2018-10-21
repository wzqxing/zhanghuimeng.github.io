---
title: Leetcode 927. Three Equal Parts（构造）
urlname: leetcode-927-three-equal-parts
toc: true
date: 2018-10-21 16:02:50
updated: 2018-10-22 00:32:50
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/three-equal-parts/description/](https://leetcode.com/problems/three-equal-parts/description/)

标记难度：Hard

提交次数：2/6

代码效率：

* 暴力set：MLE
* 枚举构造：212ms
* 直接构造：44ms

## 题意

给定一个只由0和1组成的数组，要求把数组分成三份，每一份表示的二进制数都相等。返回一种划分方法。

## 分析

比赛的时候这道题我做了好久。我先是想了一种比较暴力的方法：把数组所有的前缀去掉前导零后都放到一个`set`里面，然后再在这个`set`中查找所有的后缀是否存在，如果存在，则验证去掉前缀和后缀之后剩余的部分是否相等。结果就MLE了，因为花费的内存是`O(N^2)`。我觉得如果要维持这种做法的话，用Trie会比较好，但是我又不想写Trie。遂MLE/WA了4次。

之后我突然意识到根本没有使用`set`的必要，因为二进制数相等的前提是长度相等（去掉前导零）。只要确定了后缀去掉前导零之后的长度，就可以立即知道前缀去掉前导零之后的长度了，也即前缀的结束位置；然后判断剩余的部分去掉前导零和它们是否相等就可以了。

---

更好的方法是直接构造。为了保证三段表示的二进制数各自相等，显然需要保证每一段中1的数量相等；那么，可以先统计1的数量，如果这个数模3不为0则显然没有合法的解；否则就可以得到每一段中应有的1的数量了。然后就可以构造最后一段了：从后向前遍历数组，直到找到相应数量的1为止（因为再向前找只能找到前导0；否则1的数量就不合法了）。此时就可以知道二进制数的长度了，可以构造第一段；然后判断中间的一段和两边是否相等即可。[^prime]

[^prime]: [primeNumber's solution for Leetcode 927 - \[C++\] O(n) time, O(1) space, 12 ms with explanation & comments](https://leetcode.com/problems/three-equal-parts/discuss/183922/C++-O%28n%29-time-O%281%29-space-12-ms-with-explanation-and-comments)

## 代码

### 枚举构造

```cpp
class Solution {
private:

public:
    vector<int> threeEqualParts(vector<int>& A) {
        int n = A.size();
        string S;
        for (int x: A)
            S += to_string(x);
        int start = 0;
        // remove str1 preceeding 0
        while (S[start] == '0' && start < n) start++;
        if (start >= n) return {0, n - 1};

        for (int i = n - 1; i >= 0; i--) { // str3 = [i, n)
            int start3 = i;
            // remove str3 preceeding 0
            while (start3 < n && S[start3] == '0') start3++;
            if (start3 >= n) start3 = n - 1;
            string str3 = S.substr(start3, n - start3);
            string str1 = S.substr(start, n - start3);

            // assert str1 == str3
            if (str1 == str3) {
                // remove str2 preceeding 0
                int start2 = start + str3.size(), end2 = i;
                if (start2 >= end2) break;  // exceeded...
                while (start2 < end2 && S[start2] == '0') start2++;
                if (start2 >= end2) start2 = end2 - 1;
                string str2 = S.substr(start2, end2 - start2);
                if (str2 == str3) {
                    return {start + str3.size() - 1, i};
                }
            }
        }
        return {-1, -1};
    }
};
```

### 直接构造

```cpp
class Solution {
public:
    vector<int> threeEqualParts(vector<int>& A) {
        int ones = 0, n = A.size();
        string S;
        for (int x: A) {
            ones += x;
            S += to_string(x);
        }
        if (ones % 3 != 0) return {-1, -1};
        if (ones == 0) return {0, n - 1};

        // 构造最后一段
        int one = ones / 3;
        int start3 = n - 1, cnt = 0;
        string str3;
        for (int i = n - 1; i >= 0; i--) {
            cnt += A[i];
            if (cnt == one) {
                start3 = i;
                str3 = S.substr(i, n - i);
                break;
            }
        }
        int len = str3.size();

        // 构造第一段
        int start1 = 0;
        while (start1 < n && A[start1] == 0) start1++;
        string str1 = S.substr(start1, len);
        if (str1 != str3) return {-1, -1};

        // 构造第二段
        int start2 = start1 + len;
        while (start2 < n && A[start2] == 0) start2++;
        // 保证str2没有越过str3且中间只有0（作为str3的前导0）
        if (start2 + len > start3) return {-1, -1};
        string str2 = S.substr(start2, len);
        for (int i = start2 + len; i < start3; i++)
            if (A[i] != 0) return {-1, -1};
        if (str2 != str3) return {-1, -1};

        return {start1 + len - 1, start2 + len};
    }
};
```
