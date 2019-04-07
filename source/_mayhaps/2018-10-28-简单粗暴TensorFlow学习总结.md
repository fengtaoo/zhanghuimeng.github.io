---
title: 简单粗暴TensorFlow学习总结
urlname: a-concise-handbook-of-tensorflow
toc: true
date: 2018-10-28 21:23:15
updated: 2018-10-28 21:23:15
tags: [TensorFlow]
---

最近看了一遍[简单粗暴TensorFlow](https://tf.wiki/index.html)教程，感觉从里面学到了一些东西，不过教程的缺点也是很明显的。

## TensorFlow模型

## Eager Execution

### RNN

总之我最后发现原作者训练了60个epoch……真是恐怖的训练量啊。不过我经过计算和差不多这么多的训练之后，也输出了比较reasonable的结果：

```
diversity 0.200000:
lieve the soul and experiences of the world the superficiation of the superficiation of the soul of the superficiation of the superficiation of the speciality of the soul and the superficient of the sension of the sension of the sension of the soul to the virtue of the still the still the soul in the still the sension of the still the still propensity of the sension of the saint of the spirit of t

diversity 0.500000:
st the impose of the world and the grounds in the pain of the otim of his eternal with the world with the superficial and lough for the consint of the instinct of success of the consider of the advances with the superficial longer the suffers that his own consideration, and which recognize of a good and end with himself, the possessism of the saints of the most himself of the special of the intell

diversity 1.000000:
lieve aristed,
meally,
how yge can really good, those how
been
imposing over at the head it is it to superficiently of
thus "inclination of his philosophy--in it is does not
propeels for the word come rendere weople humanity of their rove it be a form still, in grod around firit"--perhaps not any disparger.--the "german self-inchination of all more truth in the health genite opinions of yealing of

diversity 1.200000:
gorch mest, an sympathy in quitiext
ty upon
bring of which it would imeed: fwar by a groorityice" a new;
"know on the
equal sick!; a belief! "wherether pwe-susmias who truth, in theer
is powe lifs for this own
oppossps-life, very
sungling where, every go tyde of hicd, ont he may be relaggs than nabacte exact then womied its results in, only still malt not took him should has die origing or "know m
```

## Graph Execution

总之，在尝试把这份代码的Eager Execution版本转换成Graph Execution版本的时候出现了很多问题。


### 深度强化学习（DRL）

[原版的Eager Execution](https://github.com/zhanghuimeng/learnTensorFlow/blob/master/simple_introduction/eager_execution/drl.py)运行并没有什么问题；但是，当我尝试把它改成静态版本的时候，也发生了一些错误。

```py
with tf.GradientTape() as tape:
    loss = tf.losses.mean_squared_error(        # 最小化y和Q-value的距离
        labels=y,
        predictions=tf.reduce_sum(model(tf.constant(batch_state)) *
                                  tf.one_hot(batch_action, depth=2), axis=1)
    )
grads = tape.gradient(loss, model.variables)
optimizer.apply_gradients(grads_and_vars=zip(grads, model.variables))       # 计算梯度并更新参数
```

我本来打算直接把下面这段代码挪到训练过程外面，变成：

```py
y_placeholder = tf.placeholder(name='y', shape=[None], dtype=tf.float32)
predictions_placeholder = tf.placeholder(name='predictions', shape=[None], dtype=tf.float32)
loss = tf.losses.mean_squared_error(        # 最小化y和Q-value的距离
    labels=y_placeholder,
    predictions=predictions_placeholder
)
train_op = optimizer.minimize(loss)
```

结果TensorFlow就报错了：ValueError: No variables to optimize.

事实证明是我比较sb，而这个报错非常精准。只把这两个placeholder丢到外面来完全无助于建立计算流图，至少要把`predictions`和`y`的运算过程也放到外面来才对……好吧我也不知道我是不是解释对了。因为`y`和`predictions`的计算过程依赖于`batch_state`和`batch_action`之类的东西，所以应该把它们都拿出来当成placeholder然后再建网络结构……

ssh同学表示对我这个搞法不是很理解，eager execution难道不是把TensorFlow当成了pyTorch来用了吗？正确的graph execution的写法是先把计算流图建立出来，再进行训练啊。我觉得这样说来很有道理，之前就是没有完全把计算过程抽离出来。

这说明了从eager execution开始学TensorFlow的局限性。不过在动手实践中似乎也明白了更多复杂的东西了，这说明动手是很重要的。
