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

### 什么是激励函数（Activation Function）

[这一节](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/2-6-A-activation-function/)非常简短地介绍了一下什么是激励函数。简单来说，激励函数就是把线性方程变成非线性方程。常见的激励函数包括relu、sigmoid和tanh，其中：

* 普通的神经网络：用啥都行
* 卷积神经网络：一般用relu
* 循环神经网络：一般用relu/tanh

### 激励函数（Activation Function）

这一节讲了TensorFlow中有哪些激活函数（有些我都不知道它算是激活函数）。在例子中，relu层放在第二个线性层之后，dropout层之前。[TensorFlow文档中Activation Function](https://www.tensorflow.org/api_guides/python/nn#Activation_Functions)这一节说明了现在有哪些激活函数：

* relu
* relu6
* crelu
* elu
* selu
* softplus
* softsign
* dropout
* bias_add
* sigmoid
* tanh

其中显然有些常用，有些不常用，有些（比如dropout）我都不知道它是激活函数，有些听都没听说过。

## 建造第一个神经网络

### 添加层和建造神经网络

这两节（[add_layer](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/3-1-add-layer/)和[建造神经网络](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/3-2-create-NN/)）相对比较复杂，因为它描述了一个建造完整的的神经网络的过程。

```py
import tensorflow as tf
import numpy as np
from matplotlib import pyplot as plt


def add_layer(scope, inputs, in_size, out_size, activation_function=None):
    '''Add a linear activation layer.
    :param inputs: The input data.
    :param in_size: The size of the input data.
    :param out_size: The size of the output data.
    :param activation_function: Activation function applied to the linear output.
    :return: Output data.
    '''
    # 注意这是个矩阵
    with tf.variable_scope(scope, reuse=False):
        weights = tf.get_variable('weights', [in_size, out_size], initializer=tf.random_normal_initializer(0.1))
        bias = tf.get_variable('bias', [1, out_size], initializer=tf.constant_initializer(0.05))
        outputs = tf.matmul(inputs, weights) + bias
        if not activation_function is None:
            outputs = activation_function(outputs)
        return outputs
```

在`add_layer`这个函数里，dimension（对我来说）很值得注意。事实上它输入的input的大小是batch_size \* in_size，weight矩阵的大小是in_size \* out_size，bias的大小是1 \* out_size，因此input \* weight + bias得到的矩阵大小是batch_size \* out_size（bias在相加的时候使用了[broadcasting](https://cs231n.github.io/python-numpy-tutorial/#numpy-broadcasting)技术）。

而且实际上这个函数描述的是从一层神经元输入值到另一层神经元输出值的过程——也就是说，它并不是（容易引起误解的）一层神经元，而是一层连接。

![weight实际上是线上的权重](dense-layer.png)

另一个问题是，在创建tf.Variable时到底应该使用tf.Variable()还是tf.get_variable()。两者的一个显然的差别是，tf.Variable()创建时接收的是initial-value，而tf.get_variable()接收的是initializer。我感觉initializer更加方便。以及get_variable()可以通过和variable_scope配合使用实现共享变量（在多GPU场景下很有用）。[^variable]但是这样就需要保证不同variable的名字不同（防止意外共享变量），所以需要variable_scope的名字也不同。

[^variable]: [stackoverflow - Difference between Variable and get_variable in TensorFlow](https://stackoverflow.com/questions/37098546/difference-between-variable-and-get-variable-in-tensorflow)

```py
x_data = np.random.uniform(-3, 3, [300, 1])
noise = np.random.normal(0, 0.1, [300, 1])
y_data = np.square(x_data) + 0.5 + noise
```

这一段我也写了很久，问题在于：

* numpy的API太多，记不住（面向文档的编程……）
* x_data应有的形状是竖着的（或者说是? \* batch_size * 1）……

写完之后我发现原来的代码是在\[-1, 1\]上均匀取了300个点[^linspace]，而不是像我这样直接随机取……算了，也罢了（实际效果好像也差不多）：

```py
x_data = np.linspace(-1,1,300, dtype=np.float32)[:, np.newaxis]
noise = np.random.normal(0, 0.05, x_data.shape).astype(np.float32)
y_data = np.square(x_data) - 0.5 + noise
```

[^linspace]: [numpy.linspace](https://docs.scipy.org/doc/numpy-1.15.0/reference/generated/numpy.linspace.html)

其中另一种值得注意的写法是`[:, np.newaxis]`。`np.newaxis`相当于常量`None`，它的作用是给数组增加一个新的维度，常用于将数组转换成行向量或列向量的时候。这么说可能仍然太抽象了……简单来说，`[:, np.newaxis]`会把数组变成列向量，`[:, np.newaxis]`会把数组变成行向量，如下图[^newaxis]：

![np.newaxis的作用](np-newaxis.png)

[^newaxis]: [stackoverflow - How does numpy.newaxis work and when to use it?](https://stackoverflow.com/questions/29241056/how-does-numpy-newaxis-work-and-when-to-use-it)

另一个事实是，如果`x_data`是均匀取的，则按照我原来随便写的参数（x分布在\[-3, 3\]上，noise的均值为0，标准差为0.1，y=x^2+0.5+noise，learning rate=0.1），则很容易训到爆炸，loss变成nan，需要把learning rate调整成0.05；换成练习里给的参数（x分布在\[-1, 1\]上，noise的均值为0，标准差为0.05，y=x^2-0.5+noise，learning rate=0.1）则不容易爆炸。这大概就是机器学习相关的内容了，我决定暂且不管这些，一律调整成教程里的参数。

```py
# Create graph
# x和y的每个数据都只有1维，所以第二维是1
x_placeholder = tf.placeholder(tf.float32, [None, 1], 'x')
y_placeholder = tf.placeholder(tf.float32, [None, 1], 'y')

layer1_output = add_layer('layer_1', x_placeholder, 1, 10, tf.nn.relu)
y_output = add_layer('layer_2', layer1_output, 10, 1, None)
loss = tf.reduce_mean(tf.square(y_output - y_placeholder))  # (R)MSE
# learning rate is usually < 1
train_op = tf.train.GradientDescentOptimizer(learning_rate=0.1).minimize(loss)
init = tf.global_variables_initializer()
```

这里面大部分内容之前都已经讲过了，我感觉唯一值得注意的是`loss`的写法。`tf.reduce_mean`可以直接求出Tensor沿某个轴（或者所有元素）的平均值，所以对差值的平方作`reduce_mean`就可以直接得到MSE了。再开个根号就可以得到RMSE，不过对于loss本身来说，这么做没有什么价值。（除非需要RMSE具体的值。）[^rmse-func]

[^rmse-func]: [stackoverflow - how to set rmse cost function in tensorflow](https://stackoverflow.com/questions/33846069/how-to-set-rmse-cost-function-in-tensorflow)

然而教程里是这么写的：

```py
loss = tf.reduce_mean(tf.reduce_sum(tf.square(ys - prediction), reduction_indices=[1]))
```

看起来有点怪，不过我想这应该是一种更通用（假如y的每一行有多维）的写法。当然现在`reduction_indices`这个写法已经deprecated了，换成axis比较好。

```py
# Training
with tf.Session() as sess:
    sess.run(init)
    for i in range(0, 500):
        _, loss_val = sess.run([train_op, loss], feed_dict={x_placeholder: x_data, y_placeholder: y_data})
        if i % 20 == 0:
            print("Step %d, loss = %f" % (i, loss_val))

    y_prediction = sess.run(y_output, feed_dict={x_placeholder: x_data})
```

这部分用的是真·SGD（而不是minibatch SGD），就很有趣。输出结果如下（在没爆炸的情况下）：

```
Step 0, loss = 0.199861
Step 20, loss = 0.062161
Step 40, loss = 0.026100
Step 60, loss = 0.014150
Step 80, loss = 0.010338
...
Step 460, loss = 0.004383
Step 480, loss = 0.004270
```

最后也是通过`sess.run`函数来输出最终的prediction。

```py
plt.figure()
plt.title("x and y")
plt.ylabel('y')
plt.xlabel('x')
plt.scatter(x_data, y_data, color='blue')
plt.scatter(x_data, y_prediction, color='red')
plt.show()
```

然后我还手贱直接画了一个散点图（没想到后一节的可视化不是Tensorboard而是matplotlib……）：

![红点是预测值，蓝点是真实值](random-plot.png)

似乎能从图上看出relu函数的形状……

### 更多的结果可视化

[这一节](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/3-3-visualize-result/)的核心内容是通过画线的方式展示训练过程中拟合的变化……反正还是matplotlib，而且我听说PyCharm会出一点问题，所以也不想画了。至少我从中学到了，画散点图应该用`plt.scatter`，画折线图应该用`plt.plot`这一点。

### 加速神经网络训练（Speed Up Training）

[这一节](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/3-4-A-speed-up-learning/)没有代码，主要就是（非常不详细地）介绍了几种常见的更新方法。（反映到代码里，也不过就是把GradientDescentOptimizer换成AdamOptimizer罢了；所以还是原理比较有趣。）

啊，我现在在读[An overview of gradient descent optimization algorithms](http://ruder.io/optimizing-gradient-descent/)这篇文章，但我感觉一时半会读不完了。

### 优化器（Optimizer）

[这一节](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/3-4-optimizer/)的内容看起来和上一节差不多。暂时没什么好说的了……除了他推荐了一篇[和训练神经网络相关](https://cs231n.github.io/neural-networks-3/)的文章之外。

## 可视化 - Tensorboard

这件事还挺有趣的。核心是用`writer = tf.summary.FileWriter('logs/', sess.graph)`把计算流图打出来。

第一个问题是`tf.variable_scope()`和`tf.name_scope()`有什么差别。我之前已经（手贱地）用了variable_scope，知道它可以共享Variable的权重，名称一样的variable_scope里面名称一样的Variable的值是一样的。那name_scope又是个啥呢。参考了stackoverflow的问答[^scopes]之后，我得出了这样的结论：

* variable_scope会为其中创建的所有Variable（无论是用get_variable还是直接创建）、Op和Constant的名称添加前缀，但只会为用get_variable方式创建的Variable共享权重（直接创建的话，每次都会创建一个新的Variable）
* name_scope不会为其中用get_variable方式创建的Variable的名称添加前缀（因为它默认你知道这个Variable应该在哪个variable_scope中共享权重）

[^scopes]: [stackoverflow - What's the difference of name scope and a variable scope in tensorflow?](https://stackoverflow.com/questions/35919020/whats-the-difference-of-name-scope-and-a-variable-scope-in-tensorflow)

反正这听起来还是很麻烦啊（不是那么的符合直觉）。

总之，我基本没改任何东西，直接把原图打出来了，得到了一个这样的东西：

![普通的图](tensorboard-test-1.png)

嗯，首先，gradients和GradientDescent这两个结点都不是我写的，所以把它们都丢到外面去了，不显示那么多乱七八糟的连接线（我终于明白外面的init和莫名其妙的结点都是干啥的了！）。

然后我也明白为什么把x和y放到同一个name_scope（input）下比较好了，因为这其实是很自然的（不然x和y就是散落在外面的两个小点）。layer_1和layer_2因为定义了variable_scope倒是比较好看，打开之后，发现了一堆MatMul，add和Relu之类的运算结点……（不过我觉得这还挺合理的，没必要给中间结果再包一层）。不过计算loss的过程也变成了一堆sub、square和mean之类的结点，这就不那么合理了，外面包一层叫loss比较好。

![进行了包装的图](tensorboard-test-2.png)

然后再想想，其实gradient是算梯度的过程，GradientDescent是将梯度下降应用到参数的过程，这两个不如都叫train算了。

![最后的图](tensorboard-test-3.png)