---
title: 论文：Sequence to Sequence Learning with Neural Networks
urlname: sequence-to-sequence-learning-with-neural-networks
toc: true
date: 2018-11-21 20:57:46
updated: 2018-11-21 20:57:46
tags: [Natural Language Processing, Machine Translation, Paper, Reading Report]
---

论文地址：[https://papers.nips.cc/paper/5346-sequence-to-sequence-learning-with-neural-networks.pdf](https://papers.nips.cc/paper/5346-sequence-to-sequence-learning-with-neural-networks.pdf)

这篇文章和之前的[RNN Encoder-Decoder](/post/learning-phrase-representations-using-rnn-encoder-decoder-for-statistical-machine-translation)的思路基本相同，主要区别是：

* 这篇文章中RNN隐藏单元使用的是LSTM，上一篇文章中使用的是GRU
* 这篇文章提出了把源句倒过来输入的方法，使得Encoder-Decoder方法在翻译长句的表现上提高了
* 这篇文章中生成翻译的方法是Beam Search，但上一篇文章中并没有细讲（如果我没看错的话；大概是因为他们的目标仍然是SMT）

不过这些改进目前应该已经被BiRNN+Attention方法吸收和替代了；不过这一方法目前也已经不是STOA了……

不过我在知乎上找到了一个具体的实现[^seq2seqimpl]，如果有时间尝试一下。

[^seq2seqimpl]: [基于TensorFlow框架的Seq2Seq英法机器翻译模型](https://zhuanlan.zhihu.com/p/37148308)

## 模型基本结构

>我之前完全搞不明白为什么RNN同一个batch的输入数据必须是等长的。后来查了查，发现其实真的可以不等长（tf中的[dynamic_rnn](https://www.tensorflow.org/api_docs/python/tf/nn/dynamic_rnn)）。把数据全都patch成等长可以方便输入，但是也会浪费计算时间并造成模型的不准确性。好像是这样的。

![模型基本结构](architecture.jpg)

<!--
## 实现

我照着[^seq2seqimpl]的实现写（chao）了一遍。
-->