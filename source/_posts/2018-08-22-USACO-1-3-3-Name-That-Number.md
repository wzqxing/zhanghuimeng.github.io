---
title: 'USACO 1.3.3: Name That Number'
urlname: usaco-1-3-3-name-that-number
toc: true
date: 2018-08-22 23:35:24
updated: 2018-08-23 00:24:00
tags: [USACO, alg:Hash Table]
---

## 题意

见[洛谷 P3864](https://www.luogu.org/problemnew/show/P3864)。

## 分析

简单的映射题。如果从编号出发计算所有可能的字母组合，然后再去字典里查找，那么复杂度会变得过高。题解中给出了一种利用二分查找进行优化的方法，但是不太好懂。显然把字典中的单词映射成号码，再根据号码去查找会简单（且好写）很多。

## 代码

### 我的代码

```cpp
/*
ID: zhanghu15
TASK: namenum
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <algorithm>
#include <map>

using namespace std;

int main() {
    ofstream fout("namenum.out");
    ifstream fin("namenum.in");
    ifstream fin2("dict.txt");

    map<char, char> numMap;
    numMap['A'] = numMap['B'] = numMap['C'] = '2';
    numMap['D'] = numMap['E'] = numMap['F'] = '3';
    numMap['G'] = numMap['H'] = numMap['I'] = '4';
    numMap['J'] = numMap['K'] = numMap['L'] = '5';
    numMap['M'] = numMap['N'] = numMap['O'] = '6';
    numMap['P'] = numMap['R'] = numMap['S'] = '7';
    numMap['T'] = numMap['U'] = numMap['V'] = '8';
    numMap['W'] = numMap['X'] = numMap['Y'] = '9';

    map<string, vector<string>> numToNameMap;

    // 似乎题目描述中没有明确说明，dict.txt中不会出现Q和Z？
    // 经查，确实可能出现，所以要特判
    string name, number;
    while (fin2 >> name) {
        bool isOk = true;
        number.clear();
        for (char ch: name) {
            if (ch == 'Q' || ch == 'Z') {
                isOk = false;
                break;
            }
            number += numMap[ch];
        }
        if (isOk)
            numToNameMap[number].push_back(name);
    }

    fin >> number;
    vector<string> validNames = numToNameMap[number];
    if (validNames.size() == 0)
        fout << "NONE" << endl;
    else {
        sort(validNames.begin(), validNames.end());
        for (string validName: validNames)
            fout << validName << endl;
    }

    return 0;
}
```

### 参考代码

>Here is Argentina competitor's Michel Mizrah's solution using the first method with a binary search. While it is blazingly fast, it does have the disadvantage of some fairly tricky coding in the binary search routine. A single off-by-one error would doom a program in a contest.

具体见注释。事实上，我觉得这份代码体现的思想更类似于Trie，而不是二分——我真没看出来“二分”在哪。以及我似乎找到了这份代码中的一个bug。

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char num[12],sol[12];
char dict[5000][13];
int nsolutions = 0;
int nwords;
int maxlen;
FILE *out;

/*
这个函数进行了一个枚举+二分的过程
@charloc 下一个需要枚举的编号位置
@low 当前二分下限
@high 当前二分上限
*/
void calc (int charloc, int low, int high) {
    // 枚举已经完成，判断当前二分区域中是否有符合要求的字符串
    // 我觉得在此处其实不需要用strcmp，因为前面的字母必然是相等的
    // 判断最后一个字母和长度相等即可
    // 经过一些实验，这种做法居然不对！事实上我找到了这份代码中的一个bug。
    // 在修复bug之后，这种做法就正确了。
    if (charloc == maxlen) {
        sol[charloc] = '\0';
        for (int x = low; x < high; x++) {
            // 修复bug后下列代码也是可行的：
            // if (strlen(dict[x]) == maxlen && sol[charloc-1] == dict[x][charloc-1]) {
            if (strcmp (sol, dict[x]) == 0) {
                fprintf (out, "%s\n", sol);
                nsolutions++;
            }
        }
        return;
   }
   // 根据当前枚举编号的位置进行“二分”
   // 首先找到“字典中当前位置字符与枚举结果当前位置字符相等”的下限
   // 然后遍历找到上限
   // charloc == 0时，还没有开始对字符进行枚举，所以不需要进行这一步，
   // [low, high]的范围是整个字典
   if (charloc > 0) {
        for (int j=low; j <= high; j++){
            if (sol[charloc-1] == dict[j][charloc-1]) {
                low=j;
                // 问题在于，这个while是有可能越界的。j有可能会超出high的范围
                // 但原作者没有进行判断，导致[low, high)范围内的字符串并不一定满足要求
                // 所以可能会溢出。
                // 测试样例：26678268463
                while (sol[charloc-1] == dict[j][charloc-1])
                    j++;
                high=j;
                break;
            }
            if (j == high) return;
        }
    }
    if (low > high) return;

    // 枚举下一个位置应有的字符
    switch(num[charloc]){
      case '2':sol[charloc] = 'A'; calc(charloc+1,low,high);
               sol[charloc] = 'B'; calc(charloc+1,low,high);
               sol[charloc] = 'C'; calc(charloc+1,low,high);
               break;
      case '3':sol[charloc] = 'D'; calc(charloc+1,low,high);
               sol[charloc] = 'E'; calc(charloc+1,low,high);
               sol[charloc] = 'F'; calc(charloc+1,low,high);
               break;
      case '4':sol[charloc] = 'G'; calc(charloc+1,low,high);
               sol[charloc] = 'H'; calc(charloc+1,low,high);
               sol[charloc] = 'I'; calc(charloc+1,low,high);
               break;
      case '5':sol[charloc] = 'J'; calc(charloc+1,low,high);
               sol[charloc] = 'K'; calc(charloc+1,low,high);
               sol[charloc] = 'L'; calc(charloc+1,low,high);
               break;
      case '6':sol[charloc] = 'M'; calc(charloc+1,low,high);
               sol[charloc] = 'N'; calc(charloc+1,low,high);
               sol[charloc] = 'O'; calc(charloc+1,low,high);
               break;
      case '7':sol[charloc] = 'P'; calc(charloc+1,low,high);
               sol[charloc] = 'R'; calc(charloc+1,low,high);
               sol[charloc] = 'S'; calc(charloc+1,low,high);
               break;
      case '8':sol[charloc] = 'T'; calc(charloc+1,low,high);
               sol[charloc] = 'U'; calc(charloc+1,low,high);
               sol[charloc] = 'V'; calc(charloc+1,low,high);
               break;
      case '9':sol[charloc] = 'W'; calc(charloc+1,low,high);
               sol[charloc] = 'X'; calc(charloc+1,low,high);
               sol[charloc] = 'Y'; calc(charloc+1,low,high);
               break;
   }
}

int main(){
    FILE *in=fopen ("namenum.in", "r");
    FILE *in2=fopen ("dict.txt", "r");
    int j;
    out=fopen ("namenum.out","w");
    // 输入字典
    for (nwords = 0; fscanf (in2, "%s", &dict[nwords++]) != EOF; )
        ;
    fscanf (in, "%s",&num);
    maxlen = strlen(num);
    // 从初始位置开始枚举
    calc (0, 0, nwords);
    if (nsolutions == 0) fprintf(out,"NONE\n");
    return 0;
}
```
