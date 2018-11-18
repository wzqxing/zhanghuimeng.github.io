---
title: 莫烦PYTHON教程学习记录
urlname: learning-morvanzhou-s-tensorflow-tutorial
toc: true
date: 2018-11-17 15:41:29
updated: 2018-11-17 15:41:29
tags: [TensorFlow]
---

TensorFlow真是不好学啊。

## TensorFlow基础构架

### 处理结构

这一节没有留下代码。

### 例子2

[本节](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/2-2-example2/)讲了一个非常简单的线性回归的例子，用来说明TensorFlow的结构。

首先`np.random`还是很好用的，可以生成一个`list`的值，但是需要注意值的类型。

然后我就产生了这里为什么用的是`tf.Variable`而不是`tf.placeholder`的疑问。事实上这两者区别挺大的[^variable]：

* `tf.Variable`主要用于可训练变量，在声明时必须提供初始值，在真实训练时其值是改变的
* `tf.placeholder`主要用于得到传递进来的真实的训练样本，声明时不必指定初始值，可在运行时通过`Session.run()`的`feed_dict`参数指定

[^variable]: [TensorFlow 辨异 —— tf.placeholder 与 tf.Variable](https://blog.csdn.net/lanchunhui/article/details/61712830)

然后这里是直接用的`tf.random_uniform`和`tf.zeros`，而不是`tf.random_uniform_initializer`和`tf.zeros_initializer`。事实上这两者应该都能用，不过使用方式不同[^initializer]：

* `tf.random_uniform`：返回的是一个一定大小的Tensor，Variable的大小由该Tensor指定
* `tf.random_uniform_initializer`：返回的是一个initializer，能够对某个一定大小的Tensor进行初始化，因此Variable的大小需要一开始就给定

[^initializer]: [Stackoverflow Question - What's the difference between `tf.random_normal` and `tf.random_normal_initializer`?](https://stackoverflow.com/questions/48181961/whats-the-difference-between-tf-random-normal-and-tf-random-normal-initializ)

然后这里计算`loss`的函数是对计算所得的`y`值和真实的`y`值的差作平方再取均值。这实际上就是没开方的RMSE。

然后TensorFlow还告诉我别用`tf.initialize_all_variables()`了（deprecated），换成`tf.global_variables_initializer`比较好。

```py
import tensorflow as tf
import numpy as np

x = np.random.rand(100).astype(np.float32)  # 否则它默认会用float64；然后默认用float32的tf就会报错
y_expected = x * 0.1 + 0.3

weight = tf.Variable(tf.random_uniform([1], -1, 1))
bias = tf.Variable(tf.zeros([1]))
y_calculated = x * weight + bias
loss = tf.reduce_mean(tf.square(y_calculated - y_expected))  # reduce_mean和square结合使用==
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.5)
train_op = optimizer.minimize(loss)
initializer = tf.global_variables_initializer()

with tf.Session() as sess:
    sess.run(initializer)
    for i in range(201):
        sess.run(train_op)
        if i % 20 == 0:
            print("The %dth training: " % i)
            print("weight = ", sess.run(weight))  # 需要执行一步sess.run才能打印
            print("bias = ", sess.run(bias))
```

上述代码打印出如下结果：

```
The 0th training:
weight =  [0.13367666]
bias =  [0.40867323]
The 20th training:
weight =  [0.09341485]
bias =  [0.30376434]
...
The 200th training:
weight =  [0.09999986]
bias =  [0.3000001]
```

### Session会话控制

这一节没讲什么，只讲了`tf.Session()`的两种用法：不用`with`运算符和用`with`运算符。我记得`with`有很多好处，比如可以确保资源被释放、不用close之类的，所以好像大家都用`with tf.Session() as sess`。

```py
import tensorflow as tf

matrix1 = tf.constant([[2, 2]])
matrix2 = tf.constant([[3], [3]])
product = tf.matmul(matrix1, matrix2)

### method 1
sess = tf.Session()
print("method 1:", sess.run(product))
sess.close()

### method 2
with tf.Session() as sess:
    print("method 2:", sess.run(product))
```

### Variable变量

[这一节](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/2-4-variable/)又讲了一下变量（虽然例子2里已经讲过了）。有趣的一点是这里赋值操作也作为一个operator被保存下来了。如果直接只令`counter = counter + one`然后`sess.run(counter)`并不能让它算上三次。我猜测这种写法会导致后两次run的时候，TensorFlow认为`counter`已经算完了，所以需要run赋值操作本身才能让它真的+1。

```py
import tensorflow as tf

counter = tf.Variable(0, name='counter')
one = tf.constant(1)
added = counter + one
op = tf.assign(counter, added)

init = tf.global_variables_initializer()

with tf.Session() as sess:
    sess.run(init)
    for _ in range(3):
        sess.run(op)
        print(sess.run(counter))
```

### Placeholder传入值

[这一节](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/2-5-placeholde/)讲了和Variable相对应的另一种结构，Placeholder，用来传入值。然后在这里看起来默认它的shape是None，意思大概是这是一个一维的Tensor，但是长度不定；所以你可以给它传个标量进去。（应该是这样的。）也可以给它传一个长度为1的一维Tensor，两种方法好像都能work；当然打印出来的也会从标量变成Tensor。

```py
import tensorflow as tf

x = tf.placeholder(tf.float32)
y = tf.placeholder(tf.float32)
product = tf.multiply(x, y)

with tf.Session() as sess:
    print(sess.run(product, feed_dict={x: 4, y: 5}))
```