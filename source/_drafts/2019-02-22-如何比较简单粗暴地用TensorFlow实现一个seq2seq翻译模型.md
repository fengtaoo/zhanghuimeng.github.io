---
title: 如何比较简单粗暴地用TensorFlow实现一个seq2seq翻译模型
urlname: how-to-implement-a-seq2seq-translation-model-in-a-rather-simple-and-brutal-way-using-tensorflow
toc: true
date: 2019-02-22 16:09:21
updated: 2019-02-22 16:09:21
tags: [Natural Language Processing, TensorFlow]
categories: NLP
---

之前已经讲过怎么实现一个简单粗暴的seq2seq模型了。现在不妨开始考虑实现一个更复杂一些的翻译模型。和字母模型相比，翻译模型具有以下特点：

* 需要更多预处理：包括分词、标点符号正则化等，甚至还可以做BPE
* 句子更长：可能需要把句子最大长度调整到256个词
* 训练时间更长：可能需要保存中间模型
* 需要把dev和test分开

所以写起来复杂多了。

## 数据预处理

采用的是已经经过分词、标点正则化等预处理的[WMT17 DE-EN train & dev](http://data.statmt.org/wmt17/translation-task/preprocessed/de-en/)、[WMT17 test](http://data.statmt.org/wmt17/translation-task/test.tgz)数据，数据处理参考[THUMT](https://github.com/THUNLP-MT/THUMT/blob/master/UserManual.pdf)。可以参考我之前写的[学习THUMT（2）：尝试训练一个小型模型（to be continued...）](/post/learn-thumt-2-small-model/)。

（下面将会看到一大堆各种各样的后缀，请注意别搞错了……）

下载`corpus.tc.de.gz`、`corpus.tc.en.gz`、`dev.tgz`和`test.tgz`之后，首先解压得到`corpus.tc.de`和`corpus.tc.en`（训练数据，tc即truecased之意）、`newstest201x.tc.de`和`newstest201x.tc.en`（选哪一对做dev都可以）、`newstest2017-ende-src.en.sgm`和`newstest2017-ende-ref.de.sgm`（测试数据）。用[input-from-sgm.perl](https://github.com/moses-smt/mosesdecoder/blob/master/scripts/ems/support/input-from-sgm.perl)把`newstest2017-ende-src.en.sgm`转成`newstest2017.tc.en`。

然后用训练集数据学习[BPE](https://github.com/rsennrich/subword-nmt)，并对训练集的源端和译端、开发集的源端和译端以及测试集的源端都进行BPE：

```sh
# 学习BPE词表
python subword-nmt/learn_joint_bpe_and_vocab.py \
    --input corpus.tc.de corpus.tc.en -s 32000 -o bpe32k \
    --write-vocabulary vocab.de vocab.en
# 对训练集进行BPE
python subword-nmt/apply_bpe.py --vocabulary vocab.de \
    --vocabulary-threshold 50 -c bpe32k \
    < corpus.zh > corpus.32k.de
python subword-nmt/apply_bpe.py --vocabulary vocab.en \
    --vocabulary-threshold 50 -c bpe32k \
    < corpus.en > corpus.32k.en
# 对开发集进行BPE
python subword-nmt/apply_bpe.py --vocabulary vocab.de \
    --vocabulary-threshold 50 -c bpe32k \
    < newstest2014.tc.de > newstest2014.tc.32k.de
python subword-nmt/apply_bpe.py --vocabulary vocab.en \
    --vocabulary-threshold 50 -c bpe32k \
    < newstest2014.tc.en > newstest2014.tc.32k.en
# 对测试集的源端进行BPE
python subword-nmt/apply_bpe.py --vocabulary vocab.de \
    --vocabulary-threshold 50 -c bpe32k \
    < newstest2015.tc.de > newstest2015.tc.32k.de
```

于是我们就得到了一堆`*.tc.32k.*`。用脚本[shuffle_corpus.py]()将训练数据进行shuffle：

```sh
python shuffle_corpus.py --corpus corpus.32k.de corpus.32k.en --suffix shuf
```

根据训练数据重新生成训练所需词表：

```sh
python build_vocab.py --corpus corpus.32k.de.shuf --name vocab.32k.de.txt
python build_vocab.py --corpus corpus.32k.en.shuf --name vocab.32k.en.txt
```

## 模型

### 数据处理

## 训练

## 测试