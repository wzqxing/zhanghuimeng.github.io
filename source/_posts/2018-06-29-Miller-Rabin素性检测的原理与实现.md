---
title: Miller-Rabin素性检测的原理与实现
urlname: the-principles-and-implementation-of-miller-rabin-primality-test
toc: true
mathjax: true
date: 2018-06-29 18:34:39
tags: Cryptography
---

## Miller-Rabin素性检测的原理
按维基百科的说法，这个算法似乎本来是一个依赖于广义黎曼假设的确定性算法，后来经过改造，变成了一个依赖于二次剩余理论的非确定性算法。不过这个地方我也还没有完全搞懂，再说吧。

当然，即使不从历史角度出发，也是能比较清楚地搞懂这个算法的思路来源的。下面的介绍参考了相关的维基百科。<a href="#bib1" id="bib1ref"><sup>[1]</sup></a>

首先介绍一个引理。$1^2 \bmod p$和$(-1)^2 \bmod p$总是得到1，我们称这两个数为1的“平凡平方根”。引理的内容是：当$p$为素数且$p>2$时，不存在$1 \bmod p$的“非平凡平方根”。为了证明该引理，我们假设$x$是$1 \bmod p$的平方根，于是有

$$x^2 \equiv 1 (\bmod p)$$
$$(x + 1)(x - 1) \equiv 0 (\bmod p)$$

也就是说，$(x - 1)(x + 1)$能够被素数$p$整除。根据欧几里得引理，必有$p$能够整除$x-1$或$x+1$，即$x \equiv 1 (\bmod p)$或$x \equiv -1 (\bmod p)$。因此，$1 \bmod p$的平方根均为$p$的平凡平方根，引理得证。

下面我们进行一个信仰之跃，介绍Miller-Rabin算法的思路。假设$n$是一个奇素数，且$n > 2$，此时$n - 1$必然为一个偶数，可以被表示为$2^s \times d$的形式，其中$s$和$d$均为正整数，且$d$是奇数。由费马小定理，对于一个素数$n$，有

$$a^{n-1} \equiv 1 (\bmod n)$$

因此

$$a^{2^s d} \equiv 1 (\bmod n)$$

由上述引理，我们可以对上式取平方根，得到的必然为平凡平方根1或-1。假如我们得到的平方根为1，则继续取平方根，如此循环，最后必然会得到下列两种情况之一：
1. 得到某个$0 \leq r \leq s -1$，它满足$a^{2^r d} \equiv -1 (\bmod n)$
2. 发现对于任意$0\leq r \leq s -1$，都满足$a^{2^r d} \equiv 1 (\bmod n)$，即$a^d \equiv 1 (\bmod n)$

Miller-Rabin素性检测就是上述原理的逆否：如果我们能找到这样一个$a$，使得对任意$0 \leq r \leq s - 1$满足以下两个式子：

$$a^{2^r d} \not\equiv 1 (\bmod n)$$
$$a^d \not\equiv 1 (\bmod n)$$

则$n$必然不是一个素数。我们用记号$W_n(a)$表示$a$满足上述条件，称$a$是$n$为合数的一个凭证（witness to the compositeness of n）。

## Miller-Rabin素性检测的算法伪代码

下面给出一种Miller-Rabin素性检测的算法伪代码。<a href="#bib2" id="bib2ref"><sup>[2]</sup></a>
```
Miller-Rabin(n)
    把n-1写成n-1 = 2^k * m，其中m是一个奇数
    选取随机整数a，使得1 <= a <= n-1
    b = a^m (mod n)
    if b == 1 (mod n)
        return ("n is prime")
    for i = 0 to k-1
        if b == -1 (mod n)
            return ("n is prime")
        else
            b = b^2 (mod n)
    return ("n is composite")
```

## Miller-Rabin算法的正确性
上述证明虽然说明了Miller-Rabin算法能够在某些情况下证明$n$不是一个素数，但并没有证明这个算法对于素数不会做出错误判断，也没有说明这个算法能够给出正确判断的概率。事实上，上述算法是一个**偏是**的Monte-Carlo算法，且错误概率至多为$\frac{1}{4}$。

下面首先说明什么是“偏是的Monte-Carlo算法”（定义摘自<a href="#bib2" id="bib2ref"><sup>[2]</sup></a>）。首先说明什么是判定问题和随机算法。

>一个判定问题（decision problem）是指只能回答“是（yes）”或者“否（no）”的问题。一个随机算法是指任一使用了随机数的算法（反过来，一个算法没有使用随机数，称为确定的算法）。

下面的定义属于对判定问题的随机算法。

>对于一个判定问题的一个偏是（yes-biased）Monte Carlo算法是具有下列性质的一个概率算法：一个“是”回答总是正确的，但一个“否”回答也许是不正确的。类似地，可定义一个偏否的（no-biased）Monte Carlo算法。我们说一个偏是Monte Carlo算法具有错误概率$\epsilon$，如果算法对任何回答应该为“是”的实例（instance）至多以$\epsilon$的概率给出一个不正确的回答“否”（这个概率是对于给定的输入，通过算法产生所有可能的随机选择进行计算而得出的）。

我们要回答的判定问题是这样的：给定一个正整数$n \geq 2$，问$n$是一个合数吗？

### 证明Miller-Rabin算法是一个偏是的Monte Carlo算法

使用反证法证明这一结论：假定上述算法对于某个素数$n$回答了“$n$为合数”，然后推出矛盾。由于算法回答“$n$为合数”，必有$a^m \not\equiv 1 (\bmod n)$。现在考虑在算法中检测的$b$的序列。由于$b$在`for`循环的每一步中都做平方运算，因此我们测试的值序列为$a^m, a^{2m}, a^{2^{(k-1)}m}$。由于算法回答“$n$为合数”，可知对于$0 \leq i \leq k-1$，都满足：

$$a^{2^i m} \not\equiv -1 (\bmod n)$$

由于我们假设$n$为素数，由于$n - 1 = 2^k m$，由费马小定理可知：

$$a^{2^k m} \equiv 1 (\bmod n)$$

因此$a^{2^{k-1}m}$是模$n$的1的平方根。由于$n$为素数，由引理可知，模$n$的1的平方根只有两个，即$\pm 1 \bmod n$。由算法执行过程，我们可以知道：

$$a^{2^{k-1} m} \not\equiv -1 (\bmod n)$$

由此可得

$$a^{2^{k-1} m} \equiv 1 (\bmod n)$$

因此$a^{2^{k-2}m}$是模$n$的1的平方根。以此类推，重复上述过程，我们最后可以得到：

$$a^m \equiv 1 (\bmod n)$$

但在这种情况下，算法一开始就会回答“$n$为素数”，因此推出矛盾。

好吧，其实我们并不是很需要这个证明，我只是把《密码学原理与实践》上的证明又抄了一遍而已……

### 证明Miller-Rabin的错误概率不大于1/4

Rabin在1977年的论文<a href="#bib3" id="bib3ref"><sup>[3]</sup></a>中证明了下列定理：若$n$为大约4的合数，则有

$$\frac{3(n-1)}{4} \leq c({b | W_n(b)})$$

其中$W_n(b)$的意思上面已经讲过了，表示$b$是$n$为合数的一个凭证；$c(S)$表示集合$S$中元素的数目。上述定理表明，在$[1, n-1]$的整数中，最多只有$\frac{1}{4}(n-1)$个数不是$n$为合数的凭证。因此，在$[1, n-1]$中任选整数$a$进行测试，能够测试出$n$为合数的概率$\leq \frac{3}{4}$，即Miller-Rabin算法的错误概率不大于\frac{1}{4}。（证明太长了，懒得看也懒得写了，总之论文里什么都有）

我们可能因此认为，连续对某个合数$n$进行$N$次测试，无法发现$n$为合数的概率小于$(\frac{1}{4})^N$。这个界当然是正确的，但它并不紧。事实上，有人<a href="#bib4" id="bib4ref"><sup>[4]</sup></a>证明了更紧的界：如果我们不断随机选取$k$位（二进制位）的奇数，对这个数进行$t$次互相独立的随机Miller-Rabin检测，，输出第一个通过所有测试的数，令$p_{k, t}$表示这个数是合数的概率，则$p_{k, t}$比$4^{-t}$小得多。事实上，$p_{k, t}$满足下列公式：

$k \geq 2$时，$p_{k, t} < k^2 4^{2 - \sqrt{k}}$

$t = 2, k \geq 88$或$3 \leq t \leq k/9, k \geq21$时，$p_{k, t} < k^{3/2} 2^t t^{-1/2} 4^{2 -\sqrt{tk}}$

$t \geq k/9, k \geq 21$时，$p_{k, t} < \frac{7}{20} k 2^{-5t} + \frac{1}{7} k^{15/4} 2^{-k/2 - 2t} + 12k 2^{-k/4 - 3t}$

$t \geq k/4, k \geq 21$时，$p_{k, t} < \frac{1}{7} k^{15/4} 2^{-k/2 -2t}$

## Miller-Rabin算法的实现
实际上这是今年密码学的最后一个编程作业：要求生成若干个512bit的奇数并用Miller-Rabin算法验证它们是否为素数，可以用2000以下的素数进行试除。代码见<https://gist.github.com/zhanghuimeng/e2305a5324c6705f4eba9d94f7308769>。

选择python进行程序编写是因为python自带长整型功能，很方便。但是仍然有一些需要注意的地方，比如除法的精度<a href="#bib5" id="bib5ref"><sup>[5]</sup></a>，我在这里debug了一段时间。如果需要验证自己生成的素数的正确性，可以参考这个网站<a href="#bib6" id="bib6ref"><sup>[6]</sup></a>，可以检测很大（512bit是可以的，再大没试过）的数的素性。

如果我们把这个验证问题看做是“不断生成长度为512bit的奇数并进行验证，直到得到一个素数为止；之后重新进行这一实验”的过程，则可以应用论文<a href="#bib4" id="bib4ref"><sup>[4]</sup></a>中给出的界：

$3 \leq t \leq k/9, k \geq21$时，$p_{k, t} < k^{3/2} 2^t t^{-1/2} 4^{2 -\sqrt{tk}}$

当$k = 512$时，对于几个比较小的$t$，可以计算出上述概率界：

| $t$ | $p_{k, t}$ |
| --- | ---- |
| 3 | $2.1713 \times 10^{-18}$ |
| 4 | $8.4138 \times 10^{-22}$ |
| 5 | $9.1537 \times 10^{-25}$ |
| 6 | $2.0681 \times 10^{-27}$ |
| 7 | $8.1180 \times 10^{-30}$ |
| 8 | $4.9304 \times 10^{-32}$ |
| 9 | $4.2755 \times 10^{-34}$ |
| 10 | $4.9937 \times 10^{-36}$ |
| 11 | $7.5175 \times 10^{-38}$ |
| 12 | $1.4097 \times 10^{-39}$ |
| 13 | $3.2047 \times 10^{-41}$ |

可以看出，取$n = 13$已经能够保证错误概率$< 10^{-40}$了。事实上，计算机发生随机错误的概率大约是$1.8 \times 10^{-24}$，再增加迭代次数，将Miller-Rabin算法的准确度提得更高并无意义。<a href="#bib7" id="bib7ref"><sup>[7]</sup></a>

## 参考文献
<a id="bib1" href="#bib1ref"><sup>[1]</sup></a> 米勒-拉宾素性检验. <https://zh.wikipedia.org/wiki/米勒-拉宾检验>
<a id="bib2" href="#bib2ref"><sup>[2]</sup></a> 《密码学原理与实践》第二版 P154 5.4 素性检测.
<a id="bib3" href="#bib3ref"><sup>[3]</sup></a> Probabilistic Algorithm for Testing Primality. <https://ac.els-cdn.com/0022314X80900840/1-s2.0-0022314X80900840-main.pdf?_tid=a83dac24-31c7-42a4-bb29-f9ef85871027&acdnat=1530269388_63d4d83bf5a9aebc8c2ddf333f080240>
<a id="bib4" href="#bib4ref"><sup>[4]</sup></a> Average case error estimates for the strong probable prime test. <https://www.math.dartmouth.edu/~carlp/PDF/paper88.pdf>
<a id="bib5" href="#bib5ref"><sup>[5]</sup></a> How to get the correct accuracy with big integer division in python. <https://stackoverflow.com/questions/42709746/how-to-get-the-correct-accuracy-with-big-integer-division-in-python>
<a id="bib6" href="#bib6ref"><sup>[6]</sup></a> Integer factorization calculator. <https://alpertron.com.ar/ECM.HTM>
<a id="bib7" href="#bib7ref"><sup>[7]</sup></a> RSA and prime-generator algorithms. <https://stackoverflow.com/questions/4159333/rsa-and-prime-generator-algorithms/4160517#4160517>
