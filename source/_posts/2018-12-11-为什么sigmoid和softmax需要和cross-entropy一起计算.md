---
title: 为什么sigmoid和softmax需要和cross entropy一起计算
urlname: why-we-should-compute-sigmoid-and-softmax-with-cross-entropy
toc: true
mathjax: true
date: 2018-12-11 17:21:00
updated: 2018-12-11 17:21:00
tags: [TensorFlow, Machine Learning]
---

众所周知，TensorFlow中原来是不提供单独的cross entropy loss计算函数的，只有[softmax_cross_entropy_with_logits](https://www.tensorflow.org/api_docs/python/tf/nn/softmax_cross_entropy_with_logits)和[tf.nn.sigmoid_cross_entropy_with_logits](https://www.tensorflow.org/api_docs/python/tf/nn/sigmoid_cross_entropy_with_logits)两类。（不过现在Keras里有这种东西了，[categorical_crossentropy](https://www.tensorflow.org/api_docs/python/tf/keras/backend/sparse_categorical_crossentropy)可以指明输入的是logits而非softmax）。据开发者说，这是因为：

>We provide optimized cross-entropy implementations that are fused with the softmax/sigmoid implementations because their performance and numerical stability are critical to efficient training.
>If however you are just interested in the cross entropy itself, you can compute it directly using code from the beginners tutorial:

`cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))`

>N.B. DO NOT use this code for training. Use tf.nn.softmax_cross_entropy_with_logits() instead.[^issue]

[^issue]: [TensorFlow issue - Why is there no support for directly computing cross entropy?](https://github.com/tensorflow/tensorflow/issues/2462)

那么都有些什么问题呢？

## softmax计算中的问题

softmax的公式是：

{% raw %}
$$
y_i = \frac{e^{x_i}}{\sum_{i=1}^n e^{x_i}}
$$
{% endraw %}

一个事实是，如果传入的值稍微大一些，结果就会溢出（因为指数运算的结果太大了）。解决方法是在分式上下除以一个$e^\alpha$：

{% raw %}
$$
\begin{aligned}
y_i = \frac{e^{x_i}}{\sum_{i=1}^n e^{x_i}}
= \frac{e^{x_i - \alpha}}{\sum_{i=1}^n e^{x_i - \alpha}}
\end{aligned}
$$
{% endraw %}

令$\alpha = \max{(x_1, \cdots, x_n)}$，则$x_i - \alpha \leq 0$，$e^{x_i - \alpha}$的结果趋近于0，不会发生溢出。[^zhihu]

[^zhihu]: [知乎 - Softmax函数与交叉熵](https://zhuanlan.zhihu.com/p/27223959)

结论：不要自己直接手算softmax。

## sigmoid计算中的问题

sigmoid的公式是：

$$y = \frac{1}{1 + e^{-x}}$$

这看起来还比较简单，不过仍然要注意分母溢出的问题。之前scipy的`expit`曾经出过这样的一个bug。在$x$为正数时，它计算的是$\frac{e^x}{e^x + 1}$，而python的`math.exp`在$x \geq 710$时会溢出。所以`expit(710)`也会溢出。[^numpy]

[^numpy]: [scipy issue - expit does not handle large arguments well ](https://github.com/scipy/scipy/issues/3385)

结论：最好也不要自己手算sigmoid。

## cross entropy计算中的问题

交叉熵的公式是：

$$L = -\sum_{i=1}^n y_i \log{\hat{y}_i}$$

其中$y_i$是正确（分类）结果（概率），$\hat{y}_i$是模型输出的分类概率。

一般来说，这个函数的输入都是softmax或者sigmoid之后的结果，从数学上说，可以保证在$(0, 1)$范围内；但是计算机的表示范围是有限的，很可能会出现$\hat{y}_i = 0$的情况。如果不管的话，结果就会直接溢出变成nan。所以至少要做一下预处理，把接近0的$\hat{y}_i$变成$\epsilon$之类的。

Keras的实现中还把接近1的$\hat{y}_i$变成了$1 - \epsilon$，这一点我还没想清楚为什么。[^keras]

[^keras]: [tensorflow issue - Why is there no support for directly computing cross entropy? - comment](https://github.com/tensorflow/tensorflow/issues/2462#issuecomment-300702241)

结论：也不要自己手算交叉熵。

（我之前确实遇到过nan的情况。）

## softmax + cross entropy

把softmax代入到cross entropy的公式中：

{% raw %}
$$
\begin{aligned}
L &= -\sum_{i=1}^n y_i \log{\hat{y}_i} \\
&= -\sum_{i=1}^n y_i \log{\frac{e^{x_i}}{\sum_{j=1}^n e^{x_j}}} \\
&= -\sum_{i=1}^n y_i \left(x_i - \log{\sum_{j=1}^n e^{x_j}}\right) \\
&= -\sum_{i=1}^n x_i y_i + \left(\sum_{i=1}^n y_i\right) \left(\log{\sum_{j=1}^n e^{x_j}}\right)
\end{aligned}$$
{% endraw %}

显然上式里只有$\log{\sum_{j=1}^n e^{x_j}}$会有数值稳定性问题。可以用类似的方法来处理：令$\alpha = \max{(x_1, \cdots, x_n)}$，则

{% raw %}
$$
\log{\sum_{i=1}^n e^{x_i}} = \log{\left(e^\alpha \sum_{i=1}^n e^{x_i - \alpha}\right)} = \alpha + \log{\sum_{i=1}^n e^{x_i - \alpha}}
$$
{% endraw %}

这样就可以解决直接计算$e^{x_j}$溢出的问题了。

## sigmoid + cross entropy

{% raw %}
$$
\begin{aligned}
L &= -\sum_{i=1}^n y_i \log{\hat{y}_i} \\
&= -\sum_{i=1}^n y_i \log{\frac{1}{1+e^{-x_i}}} \\
&= \sum_{i=1}^n y_i \log{(1+e^{-x_i})}
\end{aligned}$$
{% endraw %}

如果$e^{-x_i}$很大，那么不需要计算$\log{(1+e^{-x_i})}$（可能会溢出），直接用$-x_i$作为估计值。否则$e^{-x_i}$会被截断。

## 实现

<script src="https://gist.github.com/zhanghuimeng/1fcd5aa6fdf162edce921248c7376d57.js"></script>

可以看出softmax和cross entropy一起计算效果更好（如果先算出概率分布，由于计算精度的原因，很小的概率会舍入到0，然后直接增大到EPS，所以得到的结果变小了）。

sigmoid和cross entropy一起计算效果也更好，原因是类似的。

结论：TensorFlow的API这样设计是有原因的（虽然我还是觉得应该给一个算cross entropy的API），为了保证数值稳定性，应该尽量用API，不要自己写。