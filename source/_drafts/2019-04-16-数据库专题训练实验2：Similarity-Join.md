---
title: 数据库专题训练实验2：Similarity Join
urlname: advanced-database-exp-2-similarity-join
toc: true
date: 2019-04-16 16:28:01
updated: 2019-04-16 16:28:01
tags:
categories:
---

## 时间测试框架

我决定采用论文中使用的两个数据集进行测试：

* Edit Distance：[DBLP](http://dbgroup.cs.tsinghua.edu.cn/ligl/simjoin/data/dblp.tar.bz2)
* Jaccard：[Enron](http://dbgroup.cs.tsinghua.edu.cn/ligl/simjoin/data/enron.tar.bz2)

## Pass-Join

这个算法的基本框架也是filter-and-refine：先筛掉很多不可能的对，再遍历所有剩下的对，进行精确计算。

算法的主要原理可以通过下列引理说明：将$s$分成$\tau + 1$段，记ed要求的阈值为$\tau$，根据鸽巢原理，如果$s$和$r$在阈值$\tau$的范围内相似，则$r$中必然有一个子串与$s$的一段匹配。本算法中进行的是均分，即后$k=|s|-\left\lfloor\frac{|s|}{\tau+1}\right\rfloor * (\tau + 1)$段长度为$\left\lfloor\frac{|s|}{\tau+1}\right\rfloor$，前$\tau + 1 - k$段长度为$\left\lceil\frac{|s|}{\tau+1}\right\rceil$。

显然，如果$r$中完全没有匹配$s$的某一段的子串，$s$和$r$不可能相似，所以可以进行剪枝。下面描述剪枝的算法。

一些记号：

* $S_{l}$：长度为$l$的字符串集合
* $S_{l}^i$：$S_{l}$中每个字符串第$i$小段组成的集合
* $L_{l}^i$：$S_{l}^i$的倒排列表
* $L_{l}^i(\omega)$：字符串$\omega$在$L_{l}^i$中的查询结果，即第$i$个小段为$\omega$的字符串集合

PASS-JOIN主算法流程：

1. 对所有字符串按长度和字典序排序（长度优先）
2. 遍历字符串，对于当前字符串$s$，基于length filtering，在一些倒排列表$\mathcal{L}_{l}^{i} \, (|s|-\tau \leq l \leq|s|, 1 \leq i \leq \tau+1)$中查找相似的字符串
3. 