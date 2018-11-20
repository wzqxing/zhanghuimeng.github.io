---
title: Leetcode 929. Unique Email Addresses（字符串），及周赛（108）总结
urlname: leetcode-929-unique-email-addresses-and-weekly-contest-108
toc: true
date: 2018-10-28 14:29:17
updated: 2018-10-28 14:54:00
tags: [Leetcode, Leetcode Contest, alg:String]
---

题目来源：[https://leetcode.com/problems/unique-email-addresses/description/](https://leetcode.com/problems/unique-email-addresses/description/)

标记难度：Easy

提交次数：1/1

代码效率：36ms

## 题意

给定一系列email地址，求满足以下要求的不重复地址数量：

* 将email地址分成local name和domain name，形如：local-name@domain-name
* 数据保证每个地址中有且仅有一个`@`字符
* local name中的`.`字符应被忽略
* local name中的`+`字符及其后所有字符应被忽略

## 分析

我这次比赛的最终排名是62 / 3652。但事实上我把前4题都做出来的时候，live排名是三百多名，这听起来就很奇怪了。总之，这次比赛的确比较简单……以及这次比赛第一名是[clw](https://leetcode.com/cai_lw)。

---

至于这道题……随便用字符串搞一下就可以了，然后去个重，我就不仔细思考了。

## 代码

搞了个像状态机一样的东西逐字符处理；不过我觉得还是使用find和split更好。事实上我在场上又忘了字符串处理函数具体怎么用了。

```cpp
class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        set<string> uniqueEmails;
        for (string email: emails) {
            string local;
            string domain;
            int status = 0;
            // 0: not found + / @;
            // 1: found +;
            // 2: found @
            for (char x: email) {
                if (x == '+') {
                    if (status == 0)
                        status = 1;
                    else if (status == 2)
                        domain += x;
                }
                else if (x == '.') {
                    if (status == 0 || status == 1) continue;
                    if (status == 2) domain += x;
                }
                else if (x == '@')
                    status = 2;
                else {
                    if (status == 0) local += x;
                    else if (status == 2) domain += x;
                }
            }
            // cout << local + '@' + domain << endl;
            uniqueEmails.insert(local + '@' + domain);
        }
        return uniqueEmails.size();
    }
};
```
