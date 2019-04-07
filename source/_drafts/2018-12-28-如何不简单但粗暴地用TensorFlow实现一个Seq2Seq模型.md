---
title: 如何不简单但粗暴地用TensorFlow实现一个Seq2Seq模型
urlname: how-to-implement-a-seq2seq-model-in-a-not-simple-but-brutal-way-using-tensorflow
toc: true
date: 2018-12-28 21:24:52
updated: 2018-12-28 21:24:52
tags: [Natural Language Processing, TensorFlow]
---

这篇文章的主要思路和用到的数据来自[^zhihu-seq2seq]。

[^zhihu-seq2seq]: [知乎 - 从Encoder到Decoder实现Seq2Seq模型](https://zhuanlan.zhihu.com/p/27608348)

我想体验一下，不用TensorFlow提供的那些高级API，能否写出一个seq2seq模型。事实证明，确实是可以的，就是有……**一点点**……的麻烦……

本文中的代码位于[rnn-seq2seq.py](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py)，文中通过[gist-it](https://gist-it.appspot.com/)引用。使用的TensorFlow版本为1.10.0，其他库版本见[requirements.txt](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/requirements.txt)。

## 数据处理

### 数据初步处理

我的Python水平有限，所以不妨从读入文本数据开始。两个txt文件各有10000行，`letters_source.txt`中是随机的英文字符串，`letters_target.txt`中是对应的经过排序的字母。

```
letters_source.txt:   letters_target.txt:
bsaqq                 abqqs
npy                   npy
lbwuj                 bjluw
bqv                   bqv
kial                  aikl
...                   ...
```

如果直接用`f.readline()`，结果的字符串会带上结尾的`'\n'`：

```py
with open('../data/letters_source.txt') as f:
    source_list = f.readline()

Output: ['bsaqq\n', 'npy\n', 'lbwuj\n', ...]
```

所以应该用`f.read()`，然后再用`splitlines()`切分成行：

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=0:11&footer=no"></script>

```py
Output: ['bsaqq', 'npy', 'lbwuj', ...]
```

据说这种方法并不能非常高效地利用内存，因为它会一次把所有行都读进来；但显然这个表现就是我现在想要的，所以无所谓啦。[^sof-readline]

[^sof-readline]: [stackoverflow - Getting rid of \n when using .readlines()](https://stackoverflow.com/questions/15233340/getting-rid-of-n-when-using-readlines)

读入之后，首先需要创建一个单词表。在这种情况下，输入和输出的单词表是相同的，都是控制字符+字母。

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=11:17&footer=no"></script>

其中把一个`list`中的所有字符串都连接起来的方法是`''.join(list)`。[^sto-join]增加的3个控制字符分别是：

* `<pad>`：用于对同一batch中长度不等的序列进行填充
* `<eos>`：标志句子结尾（之后还会说到）
* `<go>`：标志句子开头（之后也会说到）

[^sto-join]: [stackoverflow - Convert a list of characters into a string](https://stackoverflow.com/questions/4481724/convert-a-list-of-characters-into-a-string)

然后创建两个词典，分别用于把字母转成id，以及把id转成字母，并记录控制字符的id（这样之后比较方便）：

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=17:26&footer=no"></script>

其中`enumerate`的返回值是一个`tuple(counter, object)`，可以比较方便地分配id。[^enumerate]

[^enumerate]: [Python enumerate()](https://www.programiz.com/python-programming/methods/built-in/enumerate)

然后就可以把`source`和`target`中的单词都转成id list了：

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=26:36&footer=no"></script>

然后把数据分成训练数据（80%）和测试数据（20%）：

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=36:47&footer=no"></script>

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=59:72&footer=no"></script>

其中随机取index的时候如果直接用`np.random.randint`会出现重复，在这种应用场景下，用`np.random.choice`比较好。[^sof-np-random]（里面的divide函数只是因为我想不出来lambda表达式怎么写了，实际上就是根据`indices`分一下。）

[^sof-np-random]: [Non-repetitive random number in numpy](https://stackoverflow.com/questions/8505651/non-repetitive-random-number-in-numpy)

### 更高级的数据处理

![seq2seq模型示意图](enc-dec.jpg)

这之后怎么处理数据就需要考虑seq2seq模型本身的构造了。Encoder逐个输入source端的字母，这一点没什么好说的。然后我们就得到了一个最终state，作为Decoder的输入之一。但是Decoder到底是怎么才能输出一整个句子的呢？作为一个RNN，它除了一个初始状态（也就是以Encoder终止的状态作为初态）以外，还需要一个输入才能开始运转啊。

考虑Decoder在测试状态下的情境，它除了一个初态真的什么都没有。这种时候，我们就只好造出一个控制字符`<go>`作为它此时的输入，等到它输出了一个字符，再把这个字符作为下一个输入，以此类推。但是那怎么才能知道它输出到了字符串的末尾，需要停止呢？那只好再造出一个控制字符`<eos>`，一旦输出`<eos>`，就停止继续输出。

以及，在训练阶段，为了让模型更加准确，Decoder每个阶段的输入不是它上一个阶段输出的字符，而是正确的target字符。这种技术叫做[teacher forcing](https://machinelearningmastery.com/teacher-forcing-for-recurrent-neural-networks/)。所以，如果原来的target是`[1, 2, 3, ..., len]`，则训练阶段Decoder需要的输入是`[<go>, 1, 2, 3, ..., len]`，期望的输出是`[1, 2, 3, ..., len, <eos>]`。我们需要按这个方法来对target进行处理。

现在需要注意的是，虽然上面说的都是输入一个单词的情况，但在实践中输入的是一个batch的单词，操作的是一个batch的state，输出的也是一个batch的单词。为了让输入等长，需要进行padding，这是为了方便RNN同时对一个batch的序列进行操作。当然，应当通过在运行RNN时指定batch中每个序列的长度来得到实际序列的运行结果（而不是又多输入了几次`<pad>`之后的错误结果）。（不过，我pad序列的最主要原因是，我实在找不到把一个变长序列的list放到一个Tensor里的方法。据说SparseTensor可以[^sof-var-len]，但是并没有尝试成功。……但是在实验中发现，padding可能并不完全是无用功？这个待会也会说到。）同时，为了得到（可能）不同长度的输出，也需要等待Decoder对每个例子都输出过了`<eos>`（或者到达了最大长度）。

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=73:113&footer=no"></script>

上述代码描述了从数据中取出一个batch并进行padding和添加控制字符处理的过程。我感觉每个batch中句子长度的差异并不大，所以也许把所有训练数据一起pad一下（每个batch中可能会多出一些无用的`<pad>`）会更好。不过目前的训练速度还在可以接受的范围内。

[^sof-var-len]: [stackoverflow - Working with variable-length text in Tensorflow](https://stackoverflow.com/questions/38619526/working-with-variable-length-text-in-tensorflow)

    事实上我感觉我不喜欢这个方法的主要原因是，要求输入的原始数据（还没转成id的）都化成Tensor，结果就都变成数据流图上的点了，必须run才能得到输出。

以及，输入和输出都会多加一层embedding矩阵。输入是先把字母id转换成one-hot向量，再乘上embedding，得到输出的稠密向量。输入对应的则是从稠密向量乘上embedding，再用argmax得到id。不过这一步不是在数据处理里做的，而是在模型里做的，所以先不去管它。

下面的代码把测试输入做了一下padding。因为测试输入是作为一整个Tensor输入进去的，所以这里整体做了一个padding。

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=47:58&footer=no"></script>

## 模型

或者说是在这个类的`__init__`方法里定义计算流图的过程。首先当然是把`placeholder`都建出来：

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=137:160&footer=no"></script>

我使用的超参数包括：

* `input_embedding_size = 15`
* `state_size = 100`
* `learning_rate = 1e-3`
* `max_len = 128`

这些参数主要是参考[^zhihu-seq2seq]定的，有一些小修改。

然后创建embedding，直接用[tf.layers.Dense](https://www.tensorflow.org/api_docs/python/tf/layers/Dense)就行。当然，更高级的API是[tf.nn.embedding_lookup](https://www.tensorflow.org/api_docs/python/tf/nn/embedding_lookup)。注意一共有三个embedding，Decoder输入和输出用的不是一个。

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=160:173&footer=no"></script>

### Encoder

然后创建Encoder。这里值得注意的是state的具体形状。对于[LSTMCell](https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/LSTMCell)来说，它的state默认是一个[LSTMStateTuple](https://www.tensorflow.org/api_docs/python/tf/nn/rnn_cell/LSTMStateTuple)，其中包含`c`（就是[^colah]中介绍的cell state）和`m`（就是[^colah]中介绍的hidden state）两种state，大小均为`num_units`。[^lstm-state-size]Encoder看起来没有什么高级API。

[^colah]: [Understanding LSTM Networks](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)

[^lstm-state-size]: [stackoverflow - What are c_state and m_state in Tensorflow LSTM?](https://stackoverflow.com/questions/41789133/what-are-c-state-and-m-state-in-tensorflow-lstm)

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=173:181&footer=no"></script>

下一个问题是，为什么用`dynamic_rnn()`，而不是`static_rnn()`，或者直接`__call__()`。事实上，`static_rnn()`做的事情就是每步调用一次`__call__()`，然后把RNN按照步数展开。[^oreilly]它的问题是，计算流图会变得过大，而且步数不是可变的。所以一般还是应该用`dynamic_rnn()`。[^static-rnn]

[^oreilly]: [Neural networks and deep learning by Aurélien Géron - Chapter 4. Recurrent Neural Networks](https://www.oreilly.com/library/view/neural-networks-and/9781492037354/ch04.html)

[^static-rnn]: [stackoverflow - What is the difference between static_rnn and dynamic_rnn?](https://stackoverflow.com/questions/51425803/what-is-the-difference-between-static-rnn-and-dynamic-rnn)

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=218:230&footer=no"></script>

### Decoder

然后创建Decoder（训练时），这比Encoder麻烦一些。由于teacher forcing，每一步的输入都是事先确定的，所以仍然直接用`dynamic_rnn()`就行。之后就是计算loss。由于我实在不会手算sequence loss，所以终于用了一回高级API，[tf.contrib.seq2seq.sequence_loss](https://www.tensorflow.org/api_docs/python/tf/contrib/seq2seq/sequence_loss)。然后应用`AdamOptimizer`并进行了gradient clipping，据说能解决梯度爆炸的问题。（事实上我也没测试到底有没有用……）

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=181:208&footer=no"></script>

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=231:247&footer=no"></script>

最后是Decoder（预测时），这比Encoder麻烦多了，我甚至需要手写`while_loop`。

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=208:217&footer=no"></script>

`finished`变量用于表示当前batch中哪些输出已经结束了。[tf.fill](https://www.tensorflow.org/api_docs/python/tf/fill)是一个用来创建可变大小（意思是它的大小可以由一个Tensor来表示）的`Variable`的好方法。在我印象里这一点`Tensor`应该做不到。[^tensor-size]`decoded_outputs`也是同样的道理：我们需要它的大小动态改变，每有一批新的输出都接到后面。

后面使用`while_loop`的一段看起来非常晦涩（当然，我也不希望这样）。理想是我能直接写一行`while not all finished`，但直接把`Tensor`放在条件判断当做`bool`会报错（判断是否为`None`则不会报错）：

```
TypeError: Using a `tf.Tensor` as a Python `bool` is not allowed. Use `if t is not None:` instead of `if t:` to test if a tensor is defined, and use TensorFlow ops such as tf.cond to execute subgraphs conditioned on the value of a tensor.
```

这是因为`if-else`只在建图时计算一次——也就是说，之后计算流图运行的时候，`if-else`的分支结构一直都和第一次计算结果相同——所以没法用`Tensor`作为`if-else`的条件，需要用[tf.cond](https://www.tensorflow.org/api_docs/python/tf/cond)之类的控制流运算结点完成**各种**条件分支操作……[^tf-cond]

[^tf-cond]: [stackoverflow - What's the difference between tf.cond and if-else?](https://stackoverflow.com/questions/45517940/whats-the-difference-between-tf-cond-and-if-else)

[^tensor-size]: [stackoverflow - tensorflow constant with variable size](https://stackoverflow.com/questions/35853483/tensorflow-constant-with-variable-size)

（当我第一次意识到这一点的时候，天啊，TensorFlow的计算流图真是太麻烦了！！！（所以现在的TF2.0自己都开始推eager execution了……））

那只能用[tf.while_loop](https://www.tensorflow.org/api_docs/python/tf/while_loop)了。简单来说，它做的是一件这样的事情：接收`cond`、`body`、`loop_vars`和`shape_invariants`作为参数，然后在`cond`为真时重复`body`。需要注意以下细节：

* `cond`和`body`接收的参数数量相同，这些参数的初始值由`loop_vars`指定（所以`loop_vars`中的变量数量也和`cond`和`body`接收的参数数量相同，这很平凡）。
* `cond`返回的参数数量与接收的数量相同；实际上，它们就是更新过的参数。这些参数随后会被传递给`body`，判断是否需要跳出循环。
* 如果在循环中各个参数的大小都不发生改变，则不需要管`shape_invariants`

<script src="http://gist-it.appspot.com/http://github.com/zhanghuimeng/learnTensorFlow/blob/master/morvan/rnn-seq2seq.py?slice=248:289&footer=no"></script>

Decoder的高级API是[tf.contrib.seq2seq.BasicDecoder](https://www.tensorflow.org/api_docs/python/tf/contrib/seq2seq/BasicDecoder)和[tf.contrib.seq2seq.dynamic_decode](https://www.tensorflow.org/api_docs/python/tf/contrib/seq2seq/dynamic_decode)。

## 训练和可视化