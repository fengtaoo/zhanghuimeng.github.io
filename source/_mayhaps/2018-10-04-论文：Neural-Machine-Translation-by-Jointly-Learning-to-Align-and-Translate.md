---
title: '论文：Neural Machine Translation by Jointly Learning to Align and Translate'
urlname: neural-machine-translation-by-jointly-learning-to-align-and-translate
toc: true
mathjax: true
date: 2018-10-04 18:50:41
updated: 2018-10-04 18:50:41
tags: [Natural Language Processing, Machine Learning, Paper, Reading Report]
---

论文链接：[https://arxiv.org/abs/1409.0473](https://arxiv.org/abs/1409.0473)

这篇文章在之前的[定长RNN Encoder-Decoder](learning-phrase-representations-using-rnn-encoder-decoder-for-statistical-machine-translation)的基础上进行了改进，提出了“Attention”机制——在我看来，事实上是把RNN接收输入序列中每一步的隐状态都记录下来作为中间表示了，然后decoder把所有这些中间表示加权平均一下，再进行decode。通过特殊的RNN处理，事实上得到了和输入序列长度相同数量的中间表示，且每个中间表示都着重于表示输入序列中的一个词，和人类的注意力集中方式很像，因此这一机制被称为“Attention”。同时，这一方法被称为RNNSearch，因为decoder可以被训练为自动搜索源句中那些与当前词有关的部分，也就是增加对应的中间表示的权重，这大大提高了原始的RNNencdec方法在长句上的表现。

Disclaimer：用图来表示的想法来自[Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)。

---

由于RNNSearch也是一个Encoder-Decoder类的模型，它的基本架构也是这样的：

![Encoder-Decoder类模型的基本架构](encdec_basic_architecture.png)

当然，中间部分的差异是重点。

## word embedding

word embedding是NLP中常用的一种技术，大概的意思是把词编码为长度和词汇表相等的one-hot vector，然后再把向量投影为一个更短也更具有表示能力的向量。在本文中，$K_x$表示源语言的词汇表大小，$K_y$表示目标语言的词汇表大小，$T_x$表示源句长度，$T_y$表示目标句长度，这也就意味着输入的句子是$T_x$个长度为$K_x$的one-hot vector：

{% raw %}
$$\mathbf{x} = (x_1, ..., x_{T_x}), x_i \in \mathbb{R}^{K_x}$$
{% endraw %}

论文并没有强调这一步骤，不过输入对应的word embedding矩阵是$\overline{E} \in \mathbb{R}^{m \times K_x}$，输出对应的矩阵是$E \in \mathbb{R}^{m \times K_y}$。

![word embedding之后的结果](word_embedding.png)

{% raw %}$${x'}_i = \overline{E} x_i$${% endraw %}

## Encoder

RNN有两个问题：

* RNN倾向于对更近的输入赋予更大的权重，即使使用LSTM或GRU单元进行优化，仍存在这个问题
* RNN在某个时刻的隐状态只包含已经输入的内容的信息（没有之后的），但我们希望每个中间表示都能包括整个句子的信息（即使是没有集中注意的部分）

因此作者进行了这样的改进：使用两个RNN，一个顺序读输入序列，另一个倒序读输入序列，然后把两个RNN读入每个词之后的隐状态综合起来，得到每个词对应的中间表示。这样做可以解决之前提到的两个问题：

* 这两个RNN输出隐状态时都刚好读完相同的词，其“注意力”正处在这个词附近
* 因为一个是正向读，另一个是逆向读，所以恰好包含了全句的信息

RNN$t$时刻的隐状态可以通过下式计算：

{% raw %}$$h_t = f({x'}_t, h_{t-1})$${% endraw %}
