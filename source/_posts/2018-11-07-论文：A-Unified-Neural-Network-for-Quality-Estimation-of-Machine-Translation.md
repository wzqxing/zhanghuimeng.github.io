---
title: 论文：A Unified Neural Network for Quality Estimation of Machine Translation
urlname: a-unified-neural-network-for-quality-estimation-of-machine-translation
toc: true
mathjax: true
date: 2018-11-07 16:02:57
updated: 2018-11-08 15:57:00
tags: [Natural Language Processing, Machine Translation, Quality Estimation, Paper, Reading Report]
---

论文地址：[https://www.jstage.jst.go.jp/article/transinf/E101.D/9/E101.D_2018EDL8019/_article/-char/en](https://www.jstage.jst.go.jp/article/transinf/E101.D/9/E101.D_2018EDL8019/_article/-char/en)

这篇论文描述了一种新的用于Quality Estimation的神经网络结构，在WMT18 QE Task中取得了较好的成绩（仅次于QEBrain，同样大大超过了SOTA）。

## 相关工作

传统的QE方法是把它看成是一个有监督回归/分类模型。典型的做法如QuEst：先从输入中提取feature，再根据feature用SVR进行打分。这种做法存在的问题之一是，提取feature的过程与源语言本身密切相关，因此限制了它在不同语言中的应用。

之后研究者们开始在QE中应用deep learning，可以分成两大类：

* neural-aware QE：将neural feature（如word embedding、translation condition probability、cross entropy等）集成到QE系统中，可以有效提高系统表现。
* pure neural QE：直接建立一个用于QE任务的神经网络；目前的SOTA是用两个分开的网络（RNNsearch predictor + RNN estimator）分别进行训练，然后输出结果。这种方法的表现比neural-aware QE更好。

本文中的做法是将RNNsearch和RNN组成一个整体的网络，共同进行训练。

（之后的实验结果将说明，pure neural QE > neural-aware QE > traditional QE，而在pure neural QE中，unified network又好于separated network。）

## 模型结构

![模型结构图](fig-1-mode-architecture.png)

（这篇文章把模型结构讲得比较细。）

模型分成两个主要模块：

* RNNsearch：从句对中提取quality vector
* RNN：用quality vector对翻译质量进行预测，可以看成是有监督回归任务

这两个模块共同进行训练。其中RNNsearch中间生成的context vector是$c_1, ..., c_n$，decoder RNN的隐状态是$s_0, s_1, ..., s_n$，$t_1, ..., t_j$是中间表示，可以通过下式计算：

{% raw %}
$$t_j = \tanh{(U_o s_{j-1} + V_o E y_{j-1} + C_o c_j)}$$
{% endraw %}

其中$U_o, V_o, C_o$是模型参数，$E$是目标语言的embedding矩阵。

给定输入$(x_1, ..., x_m)$，decoder将生成翻译输出$(y_1, ..., y_n)$，其中生成每个词的条件概率为：

{% raw %}
$$p(y_j | \{y_1, ..., y_{j-1}\}, x) = g(y_{j-1}, s_{j-1}, c) = \frac{\exp{(y^T_j W_o t_j)}}{\sum_{k=1}^{K_y} \exp{(y_k^T W_o t_j)}}$$
{% endraw %}

其中$W_o$是权重矩阵。（显然上式只是对$y^T_j W_o t_j$做了一个softmax。）为了通过上述条件概率对翻译质量进行描述，可以这样计算quality vector：

![计算quality vector的图示](fig-2-calc-quality-vector.png)

{% raw %}
$$q_{y_j} = [(y^T_j W_o) \odot t^T_j]^T$$
{% endraw %}

（所以这里就直接用了翻译条件概率……）

最后将这些quality vector依次输入到RNN（作者使用的是GRU单元）中，将最后一个输出作为QE得分：

{% raw %}
$$v_j = f(v_{j-1}, q_{y_j})$$
$$QE_{score} = W_{QE} \times v_n$$
{% endraw %}

其中$W_{QE}$是权重矩阵。最终分数没有用logistic sigmoid函数进行平均，而是直接进行了clip。

## 模型训练

由于QE任务的训练集太小了，因此RNNsearch和QE RNN先分别用平行语料（WMT17翻译任务的训练语料）和QE训练语料进行了预训练，然后才共同用QE训练语料进行训练。训练目标是最小化MAE：

$$J(\theta) = \frac{1}{N} \sum{n=1}^{N} |QE_{score}(x^{(n)}, y^{(n)}, \theta) - HTER^{(n)}|$$

由于模型的各部分是一起训练的，因此输出的quality vector是可以进行训练的（而不是像estimator-predictor方法中，RNNsearch的输出是固定的翻译结果），能够提取出更准确的feature。

## 实验结果

表中列出的系统包括：

* QuEst：传统QE方法
* SHEF/QUEST-EMB：neural-aware QE方法
* JXNU/Emb+RNNLM+QuEst+SNM：neural-aware QE方法
* Predictor-Estimator：pure neural QE方法
* UNQE：本文的方法

![实验结果](result.png)

分析结果可以得到以下结论：

* pure neural QE方法好于neural-aware QE方法好于传统QE方法
* UNQE方法好于Predictor-Estimator方法
* ensemble是有效的
* 对预测分数进行logistic sigmoid不如直接进行clip

## 一些想法

这篇文章的做法是直接利用RNNsearch的翻译结果（或者说生成翻译的条件概率）进行Quality Estimation，那么它和“直接用一个最强的翻译系统进行翻译然后比较翻译结果和实际翻译输出”有什么差异呢？文中也讲到，如果真的直接用一个RNNsearch系统的翻译输出作为feature然后去预测，效果是不如像这个系统这样，将RNNsearch + QE RNN共同进行训练的。我猜测原因可能包括：

1. RNNsearch还是不够好，应该尝试直接用Transformer的翻译输出作为feature进行预测，然后再将结果和QEBrain进行比较
2. MT系统内部训练时计算loss的方式（应该是cross entropy吧）和HTER打分是有差异的，因此需要额外的训练，使得它不止可以输出正确的翻译，还可以对翻译的实际得分有更好的估计

以及一个问题：HTER是对翻译质量的最好的度量方式吗？如果直接换成人类打分（像Task3和Task4那样）会怎样？
