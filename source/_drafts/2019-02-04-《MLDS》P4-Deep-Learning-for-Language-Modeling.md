---
title: '《MLDS》P4: Deep Learning for Language Modeling'
urlname: mlds-p4-deep-learning-for-language-modeling
toc: true
mathjax: true
date: 2019-02-04 17:37:03
updated: 2019-02-04 17:37:03
tags: [MLDS, Natural Language Processing, Deep Learning]
---

视频链接：[https://www.bilibili.com/video/av9770302/?p=4](https://www.bilibili.com/video/av9770302/?p=4)

讲义链接：[http://speech.ee.ntu.edu.tw/~tlkagk/courses/MLDS_2017/Lecture/RNNLM%20(v3).pdf](http://speech.ee.ntu.edu.tw/~tlkagk/courses/MLDS_2017/Lecture/RNNLM%20(v3).pdf)

这节课好像只是为作业讲了一点预备知识，所以讲得比较短，也不太详细。

主要内容：

* 传统的语言模型N-gram
* N-gram遇到的问题
* 改进1：矩阵分解（word embedding）
* 改进2：RNN

## N-gram

语言模型相当于计算一个句子可能出现的概率：

$$P(w_1, w_2, ..., w_n)$$

（语言模型这个建模看起来是比较粗糙的……但是却比较好用吧？）

以2-gram为例：

$$P(w_1, w_2, ..., w_n) = P(w_1 | \text{START}) P(w_2 | w_1) \cdots P(w_n | w_{n-1})$$

基于频率对概率进行估计：

$$P(w_i | w_j) = \frac{C(w_j w_i)}{C(w_j)}$$

## N-gram的问题

1. 数据永远在一定程度上是稀疏的，难以准确估计概率
2. 所需参数数量非常多（平方级别）

这两个问题在本质上差不多，都是参数太多导致的。

一种解决方法是加smoothing，但是直接smoothing的结果往往不太准确。

## 解决方法1：Matrix Factorization

其实我觉得这个部分讲得有一点迷……不过大概能听懂是怎么推导的就是了。

首先不妨把参数看成一个矩阵，左侧是被预测的词，上面是历史词，矩阵中的数值就是概率。

![概率矩阵](matrix.png)

显然其中大部分格子都是未见概率，只有一小部分是有值的。

“不妨”用$v^j$向量表示被预测词，用$h^j$向量表示历史词，假设这两个向量的点积可以表示概率。