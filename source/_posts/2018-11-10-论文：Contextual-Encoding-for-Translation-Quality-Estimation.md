---
title: 论文：Contextual Encoding for Translation Quality Estimation
urlname: contextual-encoding-for-translation-quality-estimation
toc: true
mathjax: true
date: 2018-11-10 20:01:24
updated: 2018-11-12 19:43:00
tags: [Natural Language Processing, Machine Translation, Quality Estimation, Paper, Reading Report]
---

论文地址：[https://arxiv.org/pdf/1809.00129.pdf](https://arxiv.org/pdf/1809.00129.pdf)

这篇文章也是WMT18 QE Task的提交系统之一，在word-level task上取得了较好的效果（虽然我觉得一部分原因是QEBrain和UNQE没有参加一部分task）。它的主要思路是在之前的一篇文章[^pushing]的基础上做了改进，用卷积层起到类似于attention的作用。这个思路的效果看起来不如一般的attention。（而且文章写得一点也不清楚，代码库也找不到，我还是不知道模型的具体结构是什么样的。）

[^pushing]: [Pushing the Limits of Translation Quality Estimation](http://www.aclweb.org/anthology/Q17-1015)

## 模型结构

![模型结构](architecture.png)

（图里的模型结构画得一点也不确切……）

这篇文章的模型是在[^pushing]的基础上改进出来的，中间多加了一层卷积。

模型的输入是三元组$\langle s, t, \mathcal{A} \rangle$，其中$s = s_1, ..., s_M$是源句，$t = t_1, ..., t_N$是译句，$\mathcal{A} \subseteq \{(m, n) | 1 \leq m \leq M, 1 \leq n \leq N\}$是alignment。

模型分成三个主要部分：

* 词和POS的embedding层
* 卷积层
* RNN和FF层

embedding层的vector是这样构成的（$\Vert$表示row-wise concatenation）：

* 记embedding的维度为$d$，令源词和译词采用同样的embedding参数，记源词的embedding为$e_{s_i}$，译词的embedding为$e_{t_j}$
* 将每个译词自己的embedding和与它对齐的源词的embedding的平均值连接在一起，得到{% raw %}$\mathbf{x}'_j = ave(e_{s_{\mathcal{A}(:, t_j)}}) \Vert e_{t_j}${% endraw %}，这是一个长度为$2d$的向量
* 将$\mathbf{x}'\_j$和$\mathbf{x}'\_{j-1}$和$\mathbf{x}'\_{j+1}$连接在一起，得到$\mathbf{x}\_j = \mathbf{x}'\_{j-1} \Vert \mathbf{x}'\_j \Vert \mathbf{x}'\_{j+1}$，这是一个长度为$6d$的向量

然后将这些vector连接在一起进行卷积（$\oplus$表示column-wise concatenation）：

* 将$\mathbf{x}\_j$连接在一起，构成一个矩阵：$\mathbf{x}\_{1:N} = \mathbf{x}\_1 \oplus \mathbf{x}\_2 ... \oplus \mathbf{x}\_N$
* 然后对上述矩阵进行一维卷积：$c_i = f(\mathbf{w} \cdot \mathbf{x}_{i:i+h-1} + b)$，得到feature$\mathbf{c} = \{c_1, c_2, ..., c_N\}$（进行了padding）
* 在不同的窗口大小下（$\mathcal{H} = \{1, 3, 5, 7\}$）各学习$n_f = 64$个feature，将这些feature连接起来，得到$C \in \mathbb{R}^{N \times |\mathcal{H}| \cdot n_f}$的卷积层输出，相当于每个词由长度为$\mathcal{H}| \cdot n_f = 256$的向量表示（这句是我猜的）
* 最后再把上述向量和译词的POS tag embedding和与译词对齐的源词的POS tag embedding连接起来（文中没有说POS tag是怎么来的，但[^pushing]中是用TurboTagger标记的；我猜可能也要进行平均）

然后对每个词对应的向量表示进行处理（似乎使用了stacked RNN的方法，我还没太搞懂）：

* 两层FF（ReLU），隐藏层大小为400
* 一层Bi-GRU，隐藏层大小为200，前向表示和后向表示连接后进行layer normalization
* 两层FF（ReLU），隐藏层大小为200
* 一层Bi-GRU，隐藏层大小为100，同样进行layer normalization
* 一层FF（ReLU），隐藏层大小为100
* 一层FF（ReLU），隐藏层大小为50

（我猜测每个词对应的网络参数是相同的？）

然后把最后一层输出的FF feature和Marmot输出的31个baseline feature连接起来，通过softmax来预测OK/BAD label。

## 实验结果

![实验结果](result.png)

这个模型取得了比较好的结果。

作者还进行了一些sensitivity analysis（调整dropout rate）和ablation analysis（删除模型中的一些部分，观察效果），具体内容就不写了。