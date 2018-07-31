---
title: Leetcode 93. Restore IP Address（搜索）
urlname: leetcode-93-restore-ip-address
toc: true
date: 2018-07-31 20:24:12
updated: 2018-07-31 20:30:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/restore-ip-addresses/description/](https://leetcode.com/problems/restore-ip-addresses/description/)

标记难度：Medium

提交次数：1/3

代码效率：9.95%

## 题意

把一个数字串用小数点分隔成4段，问共能形成多少个合法的IP地址。

## 分析

这就是一个简单的搜索题，但是需要注意两个边界条件：

* 每一段的数字必须在0-255之间
* 数字不能有前导0

以及，做题过程中大概会用到两个函数：

* [stoi](http://www.cplusplus.com/reference/string/stoi/)：`string`转`int`
* [substr(pos, len)](http://www.cplusplus.com/reference/string/string/substr/)：取`string`的子串，注意参数的含义

## 代码

我这个代码实在写得太繁琐了。与其用4层循环，还不如稍微简化一下……

```
class Solution {
public:
    vector<string> restoreIpAddresses(string s) {
        // 看起来像是简单的搜索问题。一个IP地址的一段只能是0-255，因此长度最多为3
        // 中间可以进行剪枝
        // 不知道有没有什么奇怪的边界情况
        // 果然是有的，比如不应该有前导0
        // 以及是否需要去重？
        // 事实说明不需要。
        vector<string> ans;
        int len[4], byte[4];

        // 因为模板的原因，必须cast成int？
        for (len[0] = 1; len[0] <= min(3, (int) s.length()); len[0]++) {
            byte[0] = stoi(s.substr(0, len[0]));
            if (byte[0] > 255)
                continue;
            // 考虑前导0问题
            if (len[0] > 1 && s[0] == '0')
                continue;

            for (len[1] = 1; len[1] <= min(3, (int) s.length() - len[0]); len[1]++) {
                byte[1] = stoi(s.substr(len[0], len[1]));
                if (byte[1] > 255)
                    continue;
                if (len[1] > 1 && s[len[0]] == '0')
                    continue;

                for (len[2] = 1; len[2] <= min(3, (int) s.length() - len[0] - len[1]); len[2]++) {
                    byte[2] = stoi(s.substr(len[0] + len[1], len[2]));
                    if (byte[2] > 255)
                        continue;
                    if (len[2] > 1 && s[len[0]+len[1]] == '0')
                        continue;

                    len[3] = s.length() - len[0] - len[1] - len[2];
                    // 除了要考虑位数不够的情况，也要考虑位数太多的情况，这也是不合法的
                    if (len[3] <= 0 || len[3] > 3)
                        continue;
                    byte[3] = stoi(s.substr(len[0] + len[1] + len[2], len[3]));
                    if (byte[3] > 255)
                        continue;
                    if (len[3] > 1 && s[len[0]+len[1]+len[2]] == '0')
                        continue;

                    string ip = s.substr(0, len[0]) + "." + s.substr(len[0], len[1]) + "." +
                        s.substr(len[0] + len[1], len[2]) + "." + s.substr(len[0] + len[1] + len[2], len[3]);
                    ans.push_back(ip);
                    // cout << ip << endl;
                }
            }
        }

        return ans;
    }
};
```
