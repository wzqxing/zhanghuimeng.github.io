---
title: Leetcode 49. Group Anagrams（Hash）
urlname: leetcode-49-group-anagrams
toc: true
date: 2018-08-13 20:12:31
updated: 2018-08-13 20:31:00
tags: [Leetcode, alg:Hash Table, alg:String]
---

题目来源：[https://leetcode.com/problems/group-anagrams/description/](https://leetcode.com/problems/group-anagrams/description/)

标记难度：Medium

提交次数：1/1

代码效率：24.16%

## 题意

将一个数组中的变位词（anagram）分别归类。

## 分析

总的来说是道水题，但是有许多种不同的实现方式。

### HashMap+sort+容器

* HashMap+sort
  * 这种思路非常容易想到：把每个字符串排序之后，所有变位词的结果就相同了。我们可以把排序的结果作为map的键值。这种做法的时间复杂度是`O(N * K * log(K))`，其中`K`是字符串的最大长度。
  * 值得注意的是，题目中的所有字符串都是小写字母，因此可以从排序和键值两方面进行优化。我们可以不用`std::sort`，而是用桶排序对字符串进行排序；我们甚至可以根本不排序，而是把每个字母出现的次数作为键值（这是[题解](https://leetcode.com/problems/group-anagrams/solution/)中的第二种做法）。此时复杂度可以降低到`O(N * K)`。
* 容器
  * 如何把若干个字符串存到map中的一个键值下？由于对顺序没有要求，事实上有很多容器可以选择，我看到了用`multiset`、`vector`、`list`的。

### 一些奇技淫巧

[Java beat 100%!!! use prime number](https://leetcode.com/problems/group-anagrams/discuss/19183/Java-beat-100!!!-use-prime-number)中介绍了这样的一种做法：用不同素数的幂作为hash值，即：`Hash(str) = 2^count(a) * 3^count(b) * ... * 103^count(z)`。这样我们就可以把HashMap中的hash步骤和字符串顺序统计的步骤巧妙地结合起来了，是一种非常高效的做法。

不过我仍然觉得这是一种奇技淫巧，因为：这种做法显然只适用于字符串长度比较短的情况，否则hash值估计要爆，效率也会降低，应当进行取模；但是进行取模之后，又要考虑hash冲突问题，这样就和一般的HashMap区别不是很大了。

### map+容器

其实我之前完全没有想过，map的值可以是容器。事实上显然是可以的。以及，`std::sort`是可以给`std::string`排序的。（[Sorting Characters Of A C++ String](https://stackoverflow.com/questions/9107516/sorting-characters-of-a-c-string)）

## 代码

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        map<string, vector<string>> anagrams;
        for (string str: strs) {
            string sorted(str);
            sort(sorted.begin(), sorted.end());
            anagrams[sorted].push_back(str);
        }

        vector<vector<string>> ans;
        for (const auto& i: anagrams) {
            ans.push_back(i.second);
        }
        return ans;
    }
};
```
