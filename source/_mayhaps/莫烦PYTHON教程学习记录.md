---
title: 莫烦PYTHON教程学习记录
urlname: learning-morvanzhou-s-tensorflow-tutorial
toc: true
mathjax: true
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

### 可视化 - Tensorboard - 1

* [教程地址](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/4-1-tensorboard1/)
* [我的代码地址](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/tensorboard-nn.py)

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

### 可视化 - Tensorboard - 2

* [教程地址](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/4-2-tensorboard2/)
* [我的代码地址](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/tensorboard-histogram.py)

这一部分讲了如何打印内容到TensorBoard上GRAPH之外的section。`tf.summary`下的类型好像不多，这次用到的就是scalar和histogram。这两者的主要区别是：

* histogram会输出到TensorBoard上DISTRIBUTIONS和HISTOGRAMS两个section，其中DISTRIBUTIONS里显示的是数据（变量）的分布随训练的变化，HISTOGRAMS显示的是数据本身随训练的变化
* scalar会输出到SCALARS这个section，给出一个数据（变量）随训练变化的折线图（毕竟是标量）

使用方法是`tf.summary.histogram(name, var)`或者`tf.summary.scalar(name, var)`。不过并没有那么简单，它们不会像graph一样自动打印出来，需要手动调用`tf.summary.merge_all()`函数得到一个summary结点（不过讲道理，给了一个自动merge所有summary的函数，不需要什么时候都自己手动merge[^merge-summary]，已经很方便了），然后在每若干次训练的时候用`session.run`一下这个结点，把它的值（和对应的step）插入到FileWriter中。

[^merge-summary]: [stackoverflow - How to merge not all summaries in tensorflow?](https://stackoverflow.com/questions/46655921/how-to-merge-not-all-summaries-in-tensorflow)

我的下一个问题是，tf.summary好像用的都是step，如果想要对时间而不是step进行可视化的话，需要做什么改变吗？（还没有找到这个问题的答案。）

![SCALARS](tensorboard-scalars.png)

![DISTRIBUTIONS](tensorboard-distributions.png)

![HISTOGRAMS](tensorboard-histograms.png)

可以看出变量命名出现了一些小问题：我猜想对于现在的TensorFlow，在TensorBoard中打印出来的变量名应该和程序中的前缀是一样的了（也就是说name_scope和variable_scope应该都有效，不需要再多加东西了）。

## 高阶内容

### 分类学习（Classification）

* [教程地址](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-01-classifier/)
* [我的代码地址](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/classification-mnist-my-data.py)

这一节我写了很久，不过最终发现了很多有趣的问题。

简单来说这一节讲的就是一个经典问题，MNIST。采用的网络结构是一层（是的，只有一层）前馈网络+softmax，损失函数是交叉熵。

#### 交叉熵在分类问题中的优点

第一个问题是，为什么分类问题的损失函数要用交叉熵而不是RMSE之类的？它们看起来也很合理啊。作为一个不学无术的人我先列一下交叉熵的表达式……[^wiki]

对于多分类问题：

$l_{CE} = -\sum_{i=1}^{C} p_i \log{q_i}$

其中$p$是一个长度为$C$的one-hot形式的向量，表示样本的真实分类；$q$是一个和为1的长度为$C$的向量，是模型输出的各分类的概率。

对于二分类问题，上式可以简化为：

$l_{CE} = -y\log{\hat{y}} - (1-y)\log{(1-\hat{y})}$

[^wiki]: [Wikipedia - Cross entropy](https://en.wikipedia.org/wiki/Cross_entropy)

然后我们可以举一个例子来比较不同的损失函数的效果。[^mccaffrey]

[^mccaffrey]: [James D. McCaffrey - Why You Should Use Cross-Entropy Error Instead Of Classification Error Or Mean Squared Error For Neural Network Classifier Training](https://jamesmccaffrey.wordpress.com/2013/11/05/why-you-should-use-cross-entropy-error-instead-of-classification-error-or-mean-squared-error-for-neural-network-classifier-training/)

神经网络1的输出：

| 输出1 | 目标 | 是否正确？ |
| ---- |--- | --- |
| 0.3 0.3 0.4 | 0 0 1 | yes |
| 0.3 0.4 0.3 | 0 1 0 | yes |
| 0.1 0.2 0.7 | 1 0 0 | no |

神经网络2的输出：

| 输出2 | 目标 | 是否正确？ |
| ---- |--- | --- |
| 0.1 0.2 0.7 | 0 0 1 | yes |
| 0.1 0.7 0.2 | 0 1 0 | yes |
| 0.3 0.4 0.3 | 1 0 0 | no |

虽然这两个神经网络的错误率都是相同的（1/3=0.33），但显然第二个神经网络更好一些，因为它更能区分正确和错误。交叉熵可以做到这一点。MSE虽然看起来好像可以做到这一点，但它可能太重视错误的例子了。

```
CE1 = (-ln(0.4)*1 - ln(0.4)*1 - ln(0.3)*1) / 3 = 1.38
CE2 = (-ln(0.7)*1 - ln(0.7)*1 - ln(0.3)*1) / 3 = 0.64
MSE1 = ((0.3-0)^2 + (0.3-0)^2 + (0.4-1)^2 + ... ) / 3 = 0.81
MSE2 = ((0.1-0)^2 + (0.2-0)^2 + (0.7-1)^2 + ... ) / 3 = 0.34
```

| 神经网络 | 错误率 | 交叉熵 | MSE |
| --- | --- | --- | --- |
| 1 | 0.33 | 1.38 | 0.81 |
| 2 | 0.33 | 0.64 | 0.34 |

好像另一个重要的问题是交叉熵和MSE的梯度（导数）。（但是我现在不想思考这个问题了，可以看这篇文章[^zhihu]……）

[^zhihu]: [知乎 - Softmax函数与交叉熵](https://zhuanlan.zhihu.com/p/27223959)

#### 如何实现分类问题

TensorFlow中曾经有很多读MNIST的方法，比如示例代码中用的`tensorflow.examples.tutorials.mnist.input_data`，但它们基本全都deprecated掉了（这个也因为网络原因需要自己从[http://yann.lecun.com/exdb/mnist/](http://yann.lecun.com/exdb/mnist/)下载数据集）；一个没deprecated的方法是用[tf.keras.datasets.mnist.load_data](https://www.tensorflow.org/api_docs/python/tf/keras/datasets/mnist/load_data)，但这个方法也因为网络原因没法用，最后我干脆也直接从[https://s3.amazonaws.com/img-datasets/mnist.npz](https://s3.amazonaws.com/img-datasets/mnist.npz)下载npz了，其中包含`x_train`，`y_train`，`x_test`和`y_test`四个array，其中图片的形状是`[None, 28, 28]`，label没有one-hot，数据类型全都是`uint8`……

当然，首先需要转换一下np数组的类型，可以利用`astype`方法……然后下一个问题是怎么batch。于是我发现只有np数组可以接收list作为下标寻址[^np-index]。于是我决定不把数据转成tensor。

[^np-index]: [stackoverflow - Passing a list of indices to another list in Python. Correct syntax?](https://stackoverflow.com/questions/25431850/passing-a-list-of-indices-to-another-list-in-python-correct-syntax)

然后就可以开始定义placeholder和数据流图了。我遇到的两个主要问题是：

1. 怎么算loss？
2. test的时候可以和train的时候混用（一部分）数据流图吗？

我感觉第二个问题的答案应该是可以，因为那就是张图，但是可以有不同的用法。（虽然如果分开写会更清楚？）但第一个问题困扰了我好久……

需要注意的第一点是，TensorFlow里提供的几种cross entropy API前面都带着softmax或者sigmoid的前缀，意思是，你传进去的是logits（没有进行处理的神经网络输出的概率），它在算cross entropy之前会帮你算一次softmax或sigmoid，所以传进去之前不！要！自！己！多！算！一！次！softmax！（我想这就是我自己开始时无论怎么调参正确率都只能到0.4左右的原因，算了两次softmax居然还能训练，已经很神奇了。）

那算完了softmax，自己再算一次cross entropy是否可取呢？看起来是可取的（[示例](https://github.com/MorvanZhou/tutorials/blob/master/tensorflowTUT/tf16_classification/full_code.py)里也是这么做的，看起来似乎没什么问题），但事实证明，换了一个数据集（就是我自己的数据集之后），这么做就不可取了。或者说这么做本来就不可取。

如果手算cross entropy，在我找的数据集上基本必然会得到`loss=nan`的效果，在原始数据集上偶尔会得到`loss=nan`的效果。这是因为手算的数值不稳定——比如它没有处理`log(0)`这种边界情况。换成TensorFlow的API之后，一切就都正常了。这说明，在这个问题上，千万不要自己造轮子。

下面是（看起来比较正常的输出）（但为什么准确率波动那么大？）：

```
step 50: loss=19296.697266
step 50: acc on eval set=0.811400
step 100: loss=11443.701172
step 100: acc on eval set=0.872900
step 150: loss=5943.838867
step 150: acc on eval set=0.838900
step 200: loss=8638.837891
step 200: acc on eval set=0.869200
step 250: loss=12599.865234
step 250: acc on eval set=0.812700
step 300: loss=6333.552734
step 300: acc on eval set=0.885300
step 350: loss=6557.052734
step 350: acc on eval set=0.878700
step 400: loss=2990.178467
step 400: acc on eval set=0.868300
step 450: loss=8121.486328
step 450: acc on eval set=0.819700
step 500: loss=6352.314941
step 500: acc on eval set=0.879100
step 550: loss=6118.964844
step 550: acc on eval set=0.850300
step 600: loss=5958.951172
step 600: acc on eval set=0.878000
step 650: loss=2806.342285
step 650: acc on eval set=0.897000
step 700: loss=3389.978027
step 700: acc on eval set=0.842800
step 750: loss=3163.475098
step 750: acc on eval set=0.900800
step 800: loss=4724.095703
step 800: acc on eval set=0.894200
step 850: loss=3702.073730
step 850: acc on eval set=0.882200
step 900: loss=4021.374268
step 900: acc on eval set=0.887000
step 950: loss=7927.149414
step 950: acc on eval set=0.838600
```

2018.12.7 UPDATE：之前写代码时不小心，忘了在预测的时候加`softmax`，结果加上之后准确率和波动的程度都没有什么变化。想想也是，都到了取`argmax`一步了，加不加`softmax`根本就没有什么区别……

#### 交叉熵计算的数值稳定性

我真的写了篇文章……

### 什么是过拟合（Overfitting）

* [教程地址](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-02-A-overfitting/)
* 我的代码地址：没有代码

这一节没有代码。讲了一下什么是过拟合，以及$L_1$和$L_2$正则化，对dropout做了一个预告。下面从一篇文章[^regularization]中摘抄一点对正则化的解释：

[^regularization]: [CSDN - 正则化方法：L1和L2 regularization、数据集扩增、dropout](https://blog.csdn.net/u012162613/article/details/44261657)

L2正则化就是在代价函数后面加上一个正则化项：

$$C = C_0 + \frac{\lambda}{2n} \sum_{\omega} \omega^2$$

正则化项是所有参数的平方的和的平均值，系数是$\lambda / 2$（方便求导）。

在不使用L2正则化时，$\omega$的梯度更新为：

$$\omega = \omega - \eta\frac{\partial C_0}{\partial\omega}$$

使用L2正则化后，$\omega$的梯度更新为：

$$\omega = \left(1 - \frac{\eta\lambda}{n}\right)\omega - \eta\frac{\partial C_0}{\partial\omega}$$

结果是参数$\omega$本身被衰减了。（这是我之前没有预料到的）一般来说，约束参数的范数可以减小过拟合。

L1正则化的形式类似：

$$C = C_0 + \frac{\lambda}{n} \sum_{\omega} |\omega|$$

使用L1正则化后，$\omega$的梯度更新为：

$$\omega = \omega - \frac{\eta\lambda}{n}\text{sgn}(\omega) - \eta\frac{\partial C_0}{\partial\omega}$$

结果是参数$\omega$向0靠近了。

### Dropout解决Overfitting

* [教程地址](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-02-dropout/)
* [我的代码地址](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/dropout.py)

在完成这一节的时候遇到了茫茫多的问题，以至于我花了好几天才解决……

第一个问题是数据集。教程里用的是`sklearn.datasets.load_digits`，但是（连我自己都忘了到底是为什么了）我不想用`sklearn`，遂决定直接用分类学习那一节里我自己搞的MNIST数据集。所以接下来解决dropout问题的结果肯定和示例不太相同，连造出dropout问题的方式都不太相同。

第二个问题是TensorBoard的显示问题。我最开始尝试打印到TensorBoard的时候，发现，不仅延迟非常严重，而且延迟还非常随机，而且还经常丢掉训练接近结束时的一些数据点；有时候干脆什么数据点都打不出来。查阅文档发现，如果希望确保训练过程中能及时看到数据变化，需要每次`add_summary`时候都进行[flush](https://www.tensorflow.org/api_docs/python/tf/summary/FileWriter#flush)，而且最后要把`FileWriter`关闭，防止数据点丢失。

第三个问题是同一数据流图的重复利用和`tf.summary`的`merge`问题。在同一个数据流图里，我做了以下事情：

1. 用`placeholder`存储图片和label，以及dropout的keep_prob
2. 将图片经过一个线性层，得到输出`output`
3. 通过`output`和label算出`loss`，将它加入scalar summary中
4. 通过Optimizer和`loss`得到`train_op`
5. 对`output`执行softmax，得到归一化后的概率后执行argmax，得到预测的类别`prediction`
6. 将`prediction`和label进行比较，得到预测准确度`accuracy`，将它加入scalar summary中

![计算流图](dropout-graph.jpg)

显然这个数据流图做的是两件事情，只是可以并行来做：

* 1、2、3、4步进行的是训练：实际调用`sess.run(train_op)`可以实现
* 1、2、5、6步进行的是测试：实际调用`sess.run(accuracy)`可以实现

虽然示例代码里分成了两张数据流图，但是我感觉自己做的并没有什么问题。不过，实践中遇到了这样的问题：`summary`怎么办？首先，`summary`和真实值是需要分开跑的。接受了这一点之后，在测试时不能把所有`summary`都`merge`起来就成了一个问题。训练时我们除了打印`accuracy`之外，还希望打印训练过程中的`loss`以及线性层的`weight`和`bias`，但是测试时并不需要；更重要的问题是，如果测试时也打印`loss`、`weight`和`bias`的`summary`，会不会导致不小心把测试数据也给训练了？仔细看了看图（或者说看了看代码），我感觉单纯打印这些东西不会导致实际发生训练（意思是`train_op`没有被执行），但是确实没什么意义，所以就使用了`tf.summary.merge`这个API（而不是直接全都`tf.summary.merge_all()`）。这个API的使用方法是，把每次执行`tf.summary.histogram/scalar`时的返回值记录下来，然后放在一个`list`里面去`merge`。

这个API本身没有什么大毛病，但我开始调试它的时候还没有意识到`FileWriter.flush()`的重要性，所以输出总是一会好一会不好……所以花了很久。

以及，其中我开始时忘了执行softmax就执行argmax了，但实际效果没有什么差别，想一下softmax的原理，好像这是件显然的事……不过为了连续性起见，我最后还是做了一遍softmax。

既然要看dropout，肯定得把数据分成训练集和测试集，然后每隔几步分别测试模型在**整个**训练集上和测试集上各自的准确度。我最开始没想明白这一点，直接测了minibatch-SGD中一个batch的训练数据上的准确度，结果折线图简直好像在空中蹦跳……然后我思考了一下，基于以下两种原因，干脆把SGD改成了GD：

* [示例代码](https://github.com/MorvanZhou/tutorials/blob/37de6e0d0e9291085d0876971baefb7bab00ace2/tensorflowTUT/tf17_dropout/full_code.py#L68)里用的就是GD而不是SGD
* 从直觉上来讲，SGD每次只算一个小批量的样本的梯度，这本质上是不容易造成模型在整体训练样本上的过拟合的，可能需要训练很多步才能观察出来

到这里我基本上把能解决的bug都解决了。但是这个时候训练集和测试集上的accuracy随训练步数变化的曲线看起来仍然十分离谱：

![按照教程里说的，使用GradientDescentOptimizer，lr=0.5的结果](dropout-learning-rate-0-5.png)

可以看出上图训练了十万次，但是丝毫没有要收敛的迹象，更别提过拟合了。于是我尝试了0.1的lr，结果并没有什么改善：

![lr=0.1，训练500次的结果](dropout-learning-rate-0-1.png)

（因为当时还没有解决summary flush的问题，所以是用Excel画的图）

既然数据差异很大，最后我也不想跟GD死磕了，干脆直接换成AdamOptimizer，效果惊人的好：

![AdamOptimizer训练500步，lr=0.001，橙色是训练集，蓝色是测试集](smooth-adam.png)

不过大概因为lr设置得太小了，看起来没收敛。所以我逐渐调整lr：

![AdamOptimizer训练500步，lr=0.005，橙色是训练集，蓝色是测试集](adam-0.005.png)

![AdamOptimizer训练500步，lr=0.01，橙色是训练集，蓝色是测试集](adam-0.01.png)

可以看出，lr=0.01的时候，训练集基本上已经收敛了，而且还出现了一点点过拟合的迹象。于是改成训练1000次：

![AdamOptimizer训练1000步，lr=0.01，橙色是训练集，蓝色是测试集](adam-0.01-step1000-overfit.png)

出现了非常明显的过拟合。接下来加上dropout：

![AdamOptimizer训练1000步，lr=0.01，keep_prob=0.5，橙色是训练集，蓝色是测试集](adam-0.01-1000steps-with-dropout.png)

可以发现dropout的效果包括：

* 降低了收敛时的准确度：这是为什么呢？
* 降低了过拟合的程度：这是我们所期望的
* 增加了抖动：因为每次丢弃是随机的，所以这是可以理解的

不过，事实上，我觉得我还没有那么理解dropout。为什么dropout是加在线性层输出后，激活函数前面？为什么训练时`keep_prob=0.5`，测试时`keep_prob=1`？关于后一个问题，显然有人和我抱有相同的疑问。简单来说，dropout的机制是让那些被丢弃的神经元以为自己的预测是错误的，因此减少对前一层的输出的依赖。而在测试（validate和test）时不dropout的理由是这样的[^dropout]：

* dropout是故意让一些神经元输出错误的结果。
* dropout是随机的，所以测试的结果也会变得随机，这是不可取的。

[^dropout]: [stackoverflow - Why disable dropout during validation and testing?](https://stackoverflow.com/questions/44223585/why-disable-dropout-during-validation-and-testing)

### 卷积神经网络CNN（Convolutional Neural Network）

* [教程地址](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-05-CNN3/)
* [我的代码地址](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/cnn-mnist.py)

CNN的基本原理大概是这样的：把一个有长宽和若干个color channel的图片逐渐压扁和在纵向上拉长，最后把拉长得到的一个柱状的东西再经过一些Feed Forward Network。当然这么说很不严谨。稍微科学地说一下，CNN中有两类很典型的网络层：

* 卷积层：用若干个长宽比较小，高（channel数）与输入（图片）相等的filter以一定stride给图片做卷积，输出的长宽取决于原来图片的长宽、stride大小和padding方式（但一般不会超过原来图片大小），高度等于filter数量（相当于把每个filter的结果摞起来了）
* 池化层：从输入的每一层（channel）中的每块取一个最大值，输出的长宽取决于块的大小，高度和原来相同

据说需要池化层的主要原因是，如果卷积的stride太大，会丢失信息，不如卷积时使用小stride，然后再利用池化层减小长宽。

上述内容说得还是不太科学，毫不严谨，不过就这样吧。

我的第一个问题（也是之前本来应该出现，但是因为网络结构还不太复杂被我忽略了的问题）是，既然TensorFlow已经给我们提供了[tf.layers.conv2d](https://www.tensorflow.org/api_docs/python/tf/layers/conv2d)和[tf.layers.max_pooling2d](https://www.tensorflow.org/api_docs/python/tf/layers/max_pooling2d)这两个高层API，那为啥还要用[tf.nn.conv2d](https://www.tensorflow.org/api_docs/python/tf/nn/conv2d)和[tf.nn.max_pool](ttps://www.tensorflow.org/api_docs/python/tf/nn/max_pool)呢？

这两类API的差异很简单。后一类是Op，它的功能只是做卷积/池化，没有别的了。前一类则是一整个层，它会自己帮你定义`weight`和`bias`，除了做卷积/池化以外，还会自动把`bias`给加上去（虽然池化层没有`bias`），以及其他的功能[^stackoverflow-conv2d]。我以后要是需要写代码的话，肯定尽量用高级API（[这里](https://github.com/aymericdamien/TensorFlow-Examples/blob/master/examples/3_NeuralNetworks/convolutional_network.py)是一个用较高级API写的CNN代码，很显然它写起来比较简单，有些输入输出大小都不用指定，所以我就不写了）；但是自己定义`weight`和`bias`至少有一个好处，就是我意识到了真正做卷积的时候还有个`bias`在那，这是一般的介绍里都不会提到的……

[^stackoverflow-conv2d]: [stackoverflow - difference between convolution2d and conv2d in tensorflow in terms of ussage](https://stackoverflow.com/questions/43587083/difference-between-convolution2d-and-conv2d-in-tensorflow-in-terms-of-ussage)

以及，[tf.nn.conv2d](https://www.tensorflow.org/api_docs/python/tf/nn/conv2d)和[tf.nn.max_pool](ttps://www.tensorflow.org/api_docs/python/tf/nn/max_pool)里有几个不是很直观的参数：

* `strides`：要求是长度为4的一维Tensor，且`stride[0] = stride[3] = 1`；在一般情况下，这个参数设置为`[1, stride, stride, 1]`
* `ksize`：要求是长度为4的一维Tensor，为各个维度上窗口的大小；在一般情况下也设置为`[1, size, size, 1]`

下一个问题是，既然必须自己指定输入输出大小，那么就必须搞明白每一步的Tensor和Variable的大小，不然显然是定义不出来的。

* 输入Tensor
  * `x = [batch_size, 28, 28, 1]`：图片大小为`28*28`，只有一个channel
  * `y = [batch_size]`：标签
* 卷积层1
  * `weight = [5, 5, 1, 32]`：单个filter大小为`5*5*1`，对应图片的一个channel；一共有32个filter
  * `bias = [32]`：每个filter对应一个（标量）bias
  * `output = [batch_size, 28, 28, 32]`：卷积方式为`same`，图片大小不变；channel的数量等于filter的数量
* 池化层1
  * `output = [batch_size, 14, 14, 32]`：池化大小为`2*2`，相当于图片长和宽各缩小到原来的一半，高不变
* 卷积层2
  * `weight = [5, 5, 32, 64]`：单个filter大小为`5*5*32`，对应图片的32个channel；一共有64个filter
  * `bias = [64]`：每个filter对应一个（标量）bias
  * `output = [batch_size, 14, 14, 64]`：卷积方式为`same`，图片大小不变；channel的数量等于filter的数量
* 池化层2
  * `output = [batch_size, 7, 7, 64]`：池化大小为`2*2`，相当于图片长和宽各缩小到原来的一半，高不变
* 前馈层1
  * `input = [batch_size, 7*7*64]`：用reshape拉平，作为输入
  * `weight = [7*7*64, 1024]`：输入长度*输出长度（反着定义然后反着做乘法也可以……）
  * `bias = [1024]`：输出长度
  * `output = [batch_size, 1024]`
* 前馈层2
  * `weight = [1024, 10]`：输入长度*输出长度
  * `bias = [10]`：输出长度
  * `output = [batch_size, 10]`

还有一些杂七杂八的内容。比如CNN常用的激活函数是ReLU，一般卷积层和前馈层后面都会加一层（最后一层除外，是加softmax然后得到分类结果）。比如在TensorFlow中`*`运算符对矩阵做的是element-wise乘法，普通乘法应该用`tf.matmul`。

我按照示例代码里给的参数（AdamOptimizer，lr=1e-4）训练了一下（没写dropout），发现这个lr可能还是太低了：

![lr=1e-4时的训练loss](cnn-loss-1.png)

![lr=1e-4时测试集上的准确率](cnn-acc-1.png)

准确度最高才0.8477，而且没有收敛。以及，训练中发现，再一次测完模型在训练集上的准确度会爆内存（不知道为什么，对单层可以这么做，但对CNN就不能这么做），所以干脆就不测了，反正训到测试集上收敛都有点难度。而且CNN训起来比较慢，这500步训了两三个小时。

于是我换成lr=1e-3：

![lr=1e-3时的训练loss](cnn-loss-2.png)

![lr=1e-3时测试集上的准确率](cnn-acc-2.png)

现在准确率到了0.955，虽然还不能说是特别高，不过我觉得也还可以了。以及，显然调高lr之后loss的波动变大了。

### Saver保存读取

* [教程地址](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-06-save/)
* [我的代码地址](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/saver.py)

这次的教程讲得十分简单，但是我觉得在实际应用中可能会遇到各种问题，比如`variable_scope`和`name_scope`怎么办。我们需要的大概是相同的两张图。

而且我们这次学的其实只是变量的保存和读取（需要事先创建好同一个图），而不是模型的保存和读取（不需要事先知道图的样子）。不过暂时先不管这些。TensorFlow实际上提供了比较完备的保存变量和保存模型的方法指南，如果以后需要再去看好了。[^saver-tensorflow]

[^saver-tensorflow]: [TensorFlow指南 - 保存和恢复](https://www.tensorflow.org/guide/saved_model?hl=zh-cn)

为了表现（图中）变量的保存和读取，我这回第一次用到了非默认图。用法非常简单，用`tf.Graph()`创建一张新图，然后用`with graph.as_default()`就可以在这张图中创建变量了。[^multi-graph]首先在第一张图里创建一些Variable。这也是我第一次用常量作为Variable的`initializer`，结果一直报错：

[^multi-graph]: [TensorFlow指南 - 使用多个图进行编程](https://www.tensorflow.org/guide/graphs?hl=zh-cn#programming_with_multiple_graphs)

```
ValueError: If initializer is a constant, do not specify shape
```

后来我发现，不止不要给`shape`赋值，`dtype`也不要。虽然这很合理，但是报错信息不够明确。

创建好变量之后，在这个图里开一个`tf.Session()`，运行`global_variables_initializer`之后，调用`tf.train.Saver()`创建一个Saver，直接把这个Session里的所有变量保存起来。经过尝试发现，在Windows上只需要相对路径（和TensorBoard log形成了一定的对比）。声明保存的是`.ckpt`文件，实际上得到的是一堆文件，包括`checkpoint`、`.ckpt.data-00000-of-00001`、`.ckpt.index`和`.ckpt.meta`。这到底都是什么就不去细究好了……

读入的时候再开一张图，定义好相应的变量的`shape`和`dtype`，重新开一个Session，调用`saver.restore()`恢复刚才保存的变量，在restore之前不需要初始化。

### RNN和LSTM简介

教程：

* [什么是循环神经网络RNN](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-07-A-RNN/)
* [什么是 LSTM 循环神经网络](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-07-B-LSTM/)
* [RNN 循环神经网络](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-07-RNN1/)

反正就是一堆对RNN和LSTM的简介。我觉得我还是需要拿一点比较正式的数学和形式描述出来比较好。或者说TensorFlow中这些是怎么实现的。而且搞清楚dimensionality非常重要。

在TensorFlow的实现中，LSTMCell中有两种`state`，分别是`c_state`和`m_state`。

对于第$t$个输入，LSTM会以$\mathbf{x}_t, \, \mathbf{h}_{t-1}, \, \mathbf{c}_{t-1}$作为输入，并输出$\mathbf{h}_t, \, \mathbf{c}_t$：

$$
\begin{aligned}
\mathbf{i}_t &= \sigma(W_i\mathbf{x}_t + U_i\mathbf{h}_{t-1} + \mathbf{b}_i)\\
\mathbf{f}_t &= 
\end{aligned}
$$

### RNN LSTM 循环神经网络（分类例子）

* 教程地址：[RNN LSTM 循环神经网络 (分类例子)](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-08-RNN2/)
* 代码地址：[https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-mnist.py](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-mnist.py)

这次的教程主要描述了这样一个用RNN进行图片分类的过程：

* 将每张图按行分成28个输入向量，每个向量长度为28，即输入形状为`[batch_size, time_step, input_size]`，其中`time_step=input_size=28`
* 创建一个输入隐藏层，将输入变换为`[batch_size, time_step, hidden_size]`的维度，其中`hidden_size`即LSTM单个输入的维度
* 创建batch中例子的`initial_state`，维度为`[batch_size, state_size]`，其中`state_size`即LSTM内部状态的维度（输入维度不必与内部状态维度相等）
* 将输入数据放入LSTM中，得到输出`(outputs, state)`，其中`outputs`是各个时间步对应的输出，维度为`[batch_size, time_step, state_size]`；`state=[batch_size, (c_state, m_state)]`是最后一步后的隐状态，维度均为`state_size`
* 将LSTM隐状态中的`m_state`作为预测输出（维度为`[batch_size, state_size]`），创建一个输出线性层，将它变换为`[batch_size, 10]`的形状
* 将输出做softmax后计算cross entropy loss，进行训练

听起来通俗易懂简单明了（虽然事实上可能并不是，因为变量有点多），写的时候遇到了一万个问题。其中大部分和RNN、LSTM内部实现和维度大小的问题已经放到之前去讲了，这里就不多说。

第一个问题是[tf.matmul](https://www.tensorflow.org/api_docs/python/tf/linalg/matmul)不支持广播。虽然它现在已经支持在rank>=3，除了最后两维外的维度相同，且最后两维能做乘法的情况下做矩阵乘法了，但这不是广播，我希望的是前面的维度不匹配的情况下也能广播，比如`[batch_size, time_step, input_size] * [input_size, hidden_size]`这样。结果我就只能先把前一个Tensor变换成二维的，乘完之后再变换回原来的维度。所以我还需要判断Tensor是不是二维的，变换之前还得把原来的维度记下来。这挺烦的。

第二个问题来自于`sess.run()`里的`feed_dict`。这次我把模型写成了一个类，因此在不同的函数中用了很多名字相同的Tensor，比如`x`。于是当我尝试写出类似于`feed_dict={x: x}`这样的语句的时候，TensorFlow开始报错：

```
TypeError: unhashable type: 'numpy.ndarray'
```

事实上这么写出奇的愚蠢，因为这两个`x`显然都会是你定义的数据`x`（也就是期望中的后一个`x`）。[^ndarray-feed]于是我直接把数据`x`改成了`dx`。前一个错误消失了，但我却发现模型代码里所有叫`x`的Tensor都被换成了`dx`。

[^ndarray-feed]: [stackoverflow - unhashable type: 'numpy.ndarray' error in tensorflow](https://stackoverflow.com/questions/43081403/unhashable-type-numpy-ndarray-error-in-tensorflow)

事实上这的确是标准行为，只不过我之前的代码里没有出现过Tensor名称和placeholder重复的情况。文档里是这么说的：

>If the key is a tf.Tensor, the value may be a Python scalar, string, list, or numpy ndarray that can be converted to the same dtype as that tensor. Additionally, if the key is a tf.placeholder, the shape of the value will be checked for compatibility with the placeholder.[^tf-sess-run]

[^tf-sess-run]: [TensorFlow Docs - tf.Session().run()](https://www.tensorflow.org/versions/r1.2/api_docs/python/tf/Session#run)

所以我把变量名直接换成`model.x_placeholder`和`model.y_placeholder`了，然后就好了。

还有一个比较愚蠢的问题。我之前算loss用的一直是[tf.losses.sparse_softmax_cross_entropy](https://www.tensorflow.org/api_docs/python/tf/losses/sparse_softmax_cross_entropy)，这次一时脑抽，换成了[tf.nn.sparse_softmax_cross_entropy_with_logits](https://www.tensorflow.org/api_docs/python/tf/nn/sparse_softmax_cross_entropy_with_logits)，遂发现dimension都不太对。事实上后一种计算的是每个例子的loss，前一种只是利用后一种计算的平均loss……啊，这些API真是难记。

最后一个问题是和运行RNN紧密相关的。我感觉在开始写RNN相关的代码时，就可能会开始遇到各种各样的问题……总之，在开始运行`tf.nn.dynamic_rnn`之前，通常的操作是新建一个`state`（或者说是一个batch的state），而新建`state`的时候需要提供`batch_size`：`tf.nn.rnn_cell.LSTMCell().zero_state(batch_size, dtype)`。说实话我不是很理解为什么这么设计，我感觉平时各个layer的设计都力图让我们忘掉这是一个batch，不过既然是这样，那就需要获得`batch_size`的值：既可以把它作为一个placeholder传进来，也可以用`tf.shape(x)[0]`之类的方法动态获取。[^variable-batch-dimension]

[^variable-batch-dimension]: [stackoverflow - get the size of a variable batch dimension](https://stackoverflow.com/questions/38236175/get-the-size-of-a-variable-batch-dimension)

然后就可以训练了。在尝试打印各种`Variable`的时候，我感觉也可以尝试把LSTM的`weight`之类的东西打印出来。据说[tf.nn.rnn_cell.LSTMCell.variables](https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/LSTMCell#variables)里包含了这些内容，但是事实上只有一个`bias`，一个`kernel`，也不太明白到底是LSTM里的什么部位。[^visualize-lstm]

[^visualize-lstm]: [stackoverflow - Tensorboard - visualize weights of LSTM](https://stackoverflow.com/questions/47640455/tensorboard-visualize-weights-of-lstm)

![LSTM的bias和kernel](lstm-bias-kernel.png)

训练参数是，隐藏层大小128，LSTM内部状态大小128，lr=1e-3，loss和acc如下图：

![loss](rnn-mnist-loss.png)

![accuracy](rnn-mnist-test.png)

感觉波动得相当厉害，而且最后效果也不算很好，准确率最大只有0.8840；可能是我调参水平有限，或者RNN并没有那么适合这个任务。

### RNN LSTM（回归例子）

* 教程地址：[RNN LSTM (回归例子)](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/5-09-RNN3/)
* 代码地址：

我感觉作者教程里用的例子有点奇怪。所以我打算直接用另一篇教程中的seq2seq数据[^seq2seq-zhihu]（英文单词和字母经过排序的英文单词），以及seq2seq的网络结构来尝试进行训练。

结果写死我了。。。。。。。

来，文章在这里：

### 

[^seq2seq-zhihu]: [从Encoder到Decoder实现Seq2Seq模型](https://zhuanlan.zhihu.com/p/27608348)

## 附录：在Windows上用PyCharm运行TensorFlow

想在Windows上进行真正的深度学习训练的人都是很有勇气的。我没有那么多的勇气，不过为了不成天背着两台电脑跑来跑去，我在12月中旬的时候还是决定在自己的电脑上搞一个能用的TensorFlow环境，只是为了学习TensorFlow并做一些小的验证。以后要是能在我的电脑上写代码，然后实时同步到服务器上，在服务器上跑，然后在这边还能看到跑的结果，甚至还能看到TensorBoard的结果，那就很不错了。

首先安装PyCharm Pro版本（使用学生许可证）。这一步很容易。然后设置一下背景颜色、字体什么的。

然后把自己的代码clone下来，用PyCharm打开，这时候它用的是系统默认的Python3.5。为了统一化环境，直接新建一个virtualenv环境，然后安装对应的requirements。然后问题就来了，安装失败，pip版本太低。直接

```bash
python -m pip install --upgrade pip
```

不work，会报一堆错：

```
Installing collected packages: pip
  Found existing installation: pip 10.0.1
    Uninstalling pip-10.0.1:
      Successfully uninstalled pip-10.0.1
  Rolling back uninstall of pip
Exception:
Traceback (most recent call last):
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_internal\basecommand.py", line 228, in main
    status = self.run(options, args)
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_internal\commands\install.py", line 335, in run
    use_user_site=options.use_user_site,
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_internal\req\__init__.py", line 49, in install_given_reqs
    **kwargs
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_internal\req\req_install.py", line 748, in install
    use_user_site=use_user_site, pycompile=pycompile,
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_internal\req\req_install.py", line 961, in move_wheel_files
    warn_script_location=warn_script_location,
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_internal\wheel.py", line 431, in move_wheel_files
    generated.extend(maker.make(spec))
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_vendor\distlib\scripts.py", line 403, in make
    self._make_script(entry, filenames, options=options)
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_vendor\distlib\scripts.py", line 307, in _make_script
    self._write_script(scriptnames, shebang, script, filenames, ext)
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_vendor\distlib\scripts.py", line 243, in _write_script
    launcher = self._get_launcher('t')
  File "mypath\myproject\venv\lib\site-packages\pip-10.0.1-py3.5.egg\
pip\_vendor\distlib\scripts.py", line 382, in _get_launcher
    result = finder(distlib_package).find(name).bytes
AttributeError: 'NoneType' object has no attribute 'bytes'
```

据说发生这个问题的原因是，PyCharm把pip当成一个egg包给装上去了，但是pip不支持egg安装的升级。解决方法是[^pycharmpip]

```bash
python -m pip install -U --force-reinstall pip
```

[^pycharmpip]: [pip - issues - pip 18.0 install fails with AttributeError: 'NoneType' object has no attribute 'bytes'](https://github.com/pypa/pip/issues/5820)

解决完这个问题，下一个问题是pip从官方的源下载太慢了。所以我直接在`pip install`的时候指定了源的地址：

```bash
pip  install  -i  https://pypi.doubanio.com/simple/  --trusted-host pypi.doubanio.com  tensorflow
```

或者也可以在本机和virtualenv里做永久配置，不多讲了。[^douban]说起来，那为什么Ubuntu上install requirements那么快，不都是蓝灯+校园网的配置吗……

[^douban]: [Python部落 - 豆瓣的pip源真的很快!](https://python.freelycode.com/contribution/detail/4)

最后一个问题出现在TensorBoard上。简单来说就是在Windows上，`--logdir`需要给定绝对路径，而不是执行命令目录下的相对路径。

最后，我这台电脑还是不行，跑个线性层花了十几分钟，快烧了……