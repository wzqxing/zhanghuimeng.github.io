---
title: '论文：deepQuest: A Framework for Neural-based Quality Estimation'
urlname: deepquest-a-framework-for-neural-based-quality-estimation
toc: true
mathjax: true
date: 2018-12-03 16:01:27
updated: 2018-12-03 16:01:27
tags: [Natural Language Processing, Machine Translation, Quality Estimation, Paper, Reading Report]
---

论文地址：[http://aclweb.org/anthology/C18-1266](http://aclweb.org/anthology/C18-1266)

这篇文章的主要内容是实现了一个新的叫[deepQuest](https://sheffieldnlp.github.io/deepQuest/introduction.html)的QE系统（已开源），里面包含了之前sentence-level QE的SOTA（现在已经不是了），POSTECH和他们这次新发明的Bi-RNN两个sentence-level的系统，以及在这两个sentence-level基础上的document-level QE系统，发布了一个新的[Resources for document-level Quality Estimation](https://github.com/fredblain/docQE)数据集（也已开源），并在WMT 17和上述数据集上报告了结果。结果表明，Bi-RNN在sentence-level上效果不如POSTECH（但是训练快），作为document-level的基础系统效果更好。

## 论文内容

sentence-level QE通常预测的是机器翻译输出结果的post-editing effort，但document-level QE通常是在全自动的MT应用场景下预测文档质量分数（而非PE）。目前所有的神经网络QE方法都有复杂的结构，需要预训练和手动提取feature，而且没有document-level的方法。（吐槽：事实上我觉得预训练和复杂的网络结构都是必要的，甚至是很必要的……）所以我们提出了一个新的sentence-level方法（Bi-RNN），它不需要预训练，结构很简单，训练起来很快。我们还提出了一个新的神经网络的document-level的方法，它可以将任何细粒度的QE方法的结果进行综合，得到更粗粒度的QE结果，比如将sentence-level的输出综合处理得到document-level的结果。经过在sent-level和doc-level的一些欧洲语言的SMT和NMT上输出的测试，我们发现，对高质量NMT输出进行QE的主要挑战是在已经很流利的文本中找出错误。

### Sentence-level

作者首先实现了WMT 17的冠军系统POSTECH。这个系统分成两个部分：

* predictor：encoder-decoder RNN，基于context对词进行预测，生成表示向量
* estimator：Bi-RNN，根据predictor生成的表示向量进行打分

其中predictor部分需要大量预训练。（这就和[QEBrain](/post/bilingual-expert-can-find-translation-errors)的思路基本上是一样的，只不过具体用的架构不太一样。我可能还是需要去读一下具体是怎么做的。）

然后实现了新的一种架构，Bi-RNN。这个方法的思路非常简单：

* 对输入和输出分别进行word embedding
* 分别用一个bi-RNN对输入和输出的embedding进行学习，得到一系列隐状态$h_j$（两个RNN分开训练，但是输出结果连在一起，一起进行attention）
* 对所有的隐状态进行attention，得到加权后的和
* 进行sigmoid，将输出作为分数

attention的公式为：

{% raw %}
$$
\alpha_j = \frac{\exp{(W_a h_j^T)}}{\sum_{k=1}^{J} \exp{(W_a h_j^T)}} \\
v = \sum_{j=1}^J \alpha_j h_j
$$
{% endraw %}

![Bi-RNN结构图](sent.jpg)

### Document-level

之前我们可以看到，Bi-RNN输出了一个$v$向量，实际上可以把它看做是一个对整个句子的表示。POSTECH中Predictor也会输出一个类似的向量。我们可以把这些向量表示再做一次Bi-RNN，对结果进行attention（或者直接用最后一个隐状态），得到整个文档对应的向量，再对这个向量进行sigmoid，得到文档对应的分数。

![document-level结构图](doc.jpg)

## 测试结果

### Sentence-level

测试使用的是WMT 2017 QE的官方数据的一个超集，包括NMT和SMT的翻译结果，其中包括：

* En-De：28000句（IT领域）
* En-Lv：18768句（生命科学领域）

数据使用TERCOM进行标注，分数采用HTER。POSTECH的predictor使用Europarl和WMT 2017 News translation Task的语料进行预训练。

一些实现细节（当然，代码里有更多的细节）：

* 使用Keras进行实现（而且似乎支持Theano和TensorFlow两种backend）
* 使用GRU作为RNN单元
* word embedding维度为300
* 词表大小为30K
* encoder隐藏层单元大小为50
* 用Adadelta optimizer最小化MSE

![sent-level结果](table-1.png)

可以得出以下结论：

* En-De数据集中NMT的翻译质量高于SMT，而En-Lv数据集中NMT的翻译质量不如SMT，这影响了各个方法的表现
* POSTECH方法在SMT数据上的表现比在NMT数据上高40%，这可能受到了数据质量或者句子长度的影响
* 无预训练的POSTECH方法表现不如Bi-RNN，这可能是因为Bi-RNN能够把握NMT数据的流利度
* （但是，还是有预训练的POSTECH方法效果最好）

### Document-level

预测的分数采用的是几种BLEU值：

* document-level BLEU（使用NLTK进行打分）
* wBLEU：文档中句子BLEU值的加权平均，权重是句子长度
* tBLEU：也是句子的BLEU值的加权平均，但是权重是TFIDF；对每个文档，都学习一个新的TFIDF模型，并据此计算TFIDF分数

{% raw %}
$$
\text{wBLEU}_d = \frac{\sum_{i=1}^D \text{len}(R_i)\text{BLEU}_i}{\sum_{i=1}^D \text{len}(R_i)}
$$

$$
\text{tBLEU} = \sum_{i=1}^D \text{TFIDF}_i \text{BLEU}_i
$$

{% endraw %}

在document-level的测试中，使用的数据是WMT News Task中这几年提交的机器翻译结果，作者对数据进行了一定筛选，并把相应的数据集开源了（[docQE](https://github.com/fredblain/docQE)）。

![数据集的情况](table-2.png)

document-level系统使用的是POSTECH/Bi-RNN系统输出的句子表示，并测试了使用attention/只使用最后一个状态的结果。

![测试结果](table-4.png)

通过上述结果可以发现：

* tBLEU标记下QE系统表现最好，BLEU标记下QE系统表现最差。这可能是因为tBLEU的计算方式和这种从word representation到sentence到document的结构是最类似的。
* 以Bi-RNN作为输入representation的效果整体优于POSTECH，而且训练得还快。
* attention的贡献不是统计显著的，这可能是因为相关距离会变化，很难找到最优的权重。
* 单独筛选训练数据不会提高表现。
* POSTECH非常需要预训练。
* De-En和En-Ru的预测难度最大，这可能是因为语言之间词序差异很大，相关的MT输出质量通常也比较低。

## 运行结果

TBD
