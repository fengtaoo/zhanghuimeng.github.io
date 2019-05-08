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

* $\mathcal{S}_{l}$：长度为$l$的字符串集合
* $\mathcal{S}_{l}^i$：$\mathcal{S}_{l}$中每个字符串第$i$小段组成的集合
* $\mathcal{L}_{l}^i$：$\mathcal{S}_{l}^i$的倒排列表
* $\mathcal{L}_l^i(\omega)$：字符串$\omega$在$\mathcal{L}_l^i$中的查询结果，即$\mathcal{S}_{l}$中第$i$个小段为$\omega$的字符串集合

PASS-JOIN主算法流程（self-join）：

1. 对所有字符串按长度和字典序排序（长度优先）
2. 遍历字符串，对于当前字符串$s$，基于length filtering，在一些倒排列表$\mathcal{L}_{l}^{i} \, (|s|-\tau \leq l \leq|s|, 1 \leq i \leq \tau+1)$中查找相似的字符串（这些倒排列表只包含已经访问过的字符串，这是为了防止重复）
3. 不失一般性，考虑如何在$\mathcal{L}_{l}^{i}$中查找相似的字符串
   * 执行SUBSTRING SELECTION算法流程：从$s$中选出一些子串（显然naive方法是遍历$s$的所有子串），记为$\mathcal{W}\left(s, \mathcal{L}_{l}^{i}\right)$，对于$\forall w \in \mathcal{W}\left(s, \mathcal{L}_{l}^{i}\right)$，如果$\omega$出现在$\mathcal{L}_{l}^i$中，则对于$\forall r$，$r \in \mathcal{L}_{l}^{i}(w)$，$\langle r, s\rangle$可作为候选对
   * 执行VERIFICATION算法流程：对每个候选对$\langle r, s\rangle$，计算两者的编辑距离
4. 将$s$分成$\tau + 1$个小段，插入到对应的倒排列表$\mathcal{L}_{|s|}^{i} \, (1 \leq i \leq \tau+1)$中。（因为$s$的长度是递增的，所以此时可以删除满足$k < |s| - \tau$的倒排列表$\mathcal{L}_{k}^{i}$。）

PASS-JOIN主算法流程（$\mathcal{R}$和$\mathcal{S}$）：

1. 将$\mathcal{R}$和$\mathcal{S}$中的字符串分别按长度和字典序排序
2. 将$\mathcal{S}$中的全部字符串按照上述方法创建倒排列表
3. 遍历$\mathcal{R}$中的字符串
   * 对于长度为$|r|$的字符串，在$[|r|-\tau,|r|+\tau]$长度范围内对应的倒排列表中查找相似的字符串
   * 删除长度小于$|r| - \tau$的字符串对应的倒排列表

### SUBSTRING SELECTION的改进

论文里这部分写的好像只针对self-join的情况。。。

记$\mathcal{W}(s, l)=\cup_{i=1}^{\tau+1} \mathcal{W}\left(s, \mathcal{L}_{l}^{i}\right)$（$|s| - \tau \leq l \leq |s|$），我们只使用这些子串进行SUBSTRING SELECTION。（如果不是self-join，应该是$|s| - \tau \leq l \leq |s| + \tau$。）

最简单的方法是把$s$的所有子串都放到$\mathcal{W}(s, l)$中，则子串总数为$\sum_{i=1}^{|s|}(|s|-i+1)=\frac{|s| *(|s|+1)}{2}$，比较昂贵。

下面着重于优化$\mathcal{W}\left(s, \mathcal{L}_{l}^{i}\right)$。

#### 单纯基于长度的方法

显然$\mathcal{L}_{l}^{i}$中索引的小段长度都相同（不妨记为$l_i$），显然可以只选择$s$中长度为$l_i$的子串。记这种选择方法为$\mathcal{W}_l\left(s, \mathcal{L}_{l}^{i}\right)$。

#### 基于长度和位移的方法

$\mathcal{L}_{l}^{i}$中索引的小段除了长度相同之外，在原字符串中的起始位置也相同（不妨记为$p_i$），我们可以只选择$s$中起始位置在$\left[p_{i}-\tau, p_{i}+\tau\right]$内且长度为$l_i$的子串。记这种选择方法为$\mathcal{W}_{f}\left(s, \mathcal{L}_{l}^{i}\right)$。

这种方法的主要原理是忽略了一些重复。

#### 基于位移和长度差的方法

考虑$\mathcal{L}_{l}^{i}$中与$s$匹配的字符串$r$，假设它在$p_i$处长度为$l_i$的小段与$s$在$p$处长度相同的小段匹配。记$s(r)$在匹配处左边的子串为$s_l(r_l)$，匹配处的子串为$s_m(r_m)$，右边的子串为$s_r(r_r)$。假定$s$和$r$在$s_m = r_m$处对齐，记$d_{l}=\mathrm{ED}\left(r_{l}, s_{l}\right)$，$d_{r}=\mathrm{ED}\left(r_{r}, s_{r}\right)$，为使$s$和$r$相似，必有$d_{l}+d_{r} \leq \tau$。

---

在$l \leq |s|$的情况下，记$\Delta = |s| - |r|$，$\Delta_l = p_i - p$，$\Delta_r = p - p_i$。

当$p \leq p_i$时，如下图：

![p <= pi](position-aware-min.png)

$$d_l \geq \Delta_l$$

$$d_r \geq \Delta_l + \Delta$$

显然有

$$\tau \geq d_l + d_r \geq 2\Delta_l + \Delta$$

即

$$\Delta_l \leq \frac{\tau - \Delta}{2}$$

$$p = p_i - \Delta_l \geq p_i - \left\lfloor\frac{\tau-\Delta}{2}\right\rfloor$$

当$p > p_i$时，如下图：

![p > pi](position-aware-max.png)

$$d_l \geq \Delta_r$$

$$d_r \geq | \Delta_r - \Delta |$$

如果$\Delta_r \leq \Delta$，则$d_r \geq \Delta - \Delta_r$，因此$\Delta = \Delta_r + (\Delta - \Delta_r) \leq d_l + d_r \leq \tau$；否则$\Delta_r > \Delta$，$d_r \geq \Delta_r - \Delta$，$\tau \geq d_l + d_r \geq \Delta_r + (\Delta_r - \Delta)$，即$\Delta_{r} \leq\left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$。在$|s| - \tau \leq l \leq |s| + \tau$的前提下（这是length filter），显然有$\Delta \leq \tau$，$\Delta \leq \left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$，因此这两种情况可以统一为$\Delta_r \leq \left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$，即$p=p_{i}+\Delta_{r} \leq p_{i}+\left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$。

因此有

$$
\begin{aligned}
p_{\min } &= \max \left(1, p_{i}-\left\lfloor\frac{\tau-\Delta}{2}\right\rfloor\right) \\
p_{\max } &= \min \left(|\mathrm{s}|-l_{i}+1, p_{i}+\frac{\tau+\Delta}{2}\right)
\end{aligned}
$$

---

在$|s| < l$的情况下，记$\Delta = |r| - |s|$，$\Delta_l = p_i - p$，$\Delta_r = p - p_i$。

当$p \leq p_i$时，如下图：

![p <= pi]()

$$d_l \geq \Delta_l$$

$$d_r \geq |\Delta_l - \Delta|$$

类似地，可以得到

$$p \geq p_i - \left\lfloor\frac{\tau + \Delta}{2}\right\rfloor$$

当$p > p_i$时，如下图：

![p > pi]()

$$d_l \geq \Delta_r$$

$$d_r \geq | \Delta_r - \Delta |$$

如果$\Delta_r \leq \Delta$，则$d_r \geq \Delta - \Delta_r$，因此$\Delta = \Delta_r + (\Delta - \Delta_r) \leq d_l + d_r \leq \tau$；否则$\Delta_r > \Delta$，$d_r \geq \Delta_r - \Delta$，$\tau \geq d_l + d_r \geq \Delta_r + (\Delta_r - \Delta)$，即$\Delta_{r} \leq\left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$。在$|s| - \tau \leq l \leq |s| + \tau$的前提下（这是length filter），显然有$\Delta \leq \tau$，$\Delta \leq \left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$，因此这两种情况可以统一为$\Delta_r \leq \left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$，即$p=p_{i}+\Delta_{r} \leq p_{i}+\left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$。

因此有

$$
\begin{aligned}
p_{\min } &= \max \left(1, p_{i}-\left\lfloor\frac{\tau-\Delta}{2}\right\rfloor\right) \\
p_{\max } &= \min \left(|\mathrm{s}|-l_{i}+1, p_{i}+\frac{\tau+\Delta}{2}\right)
\end{aligned}
$$

我们可以只选择长度为$l_i$且起始位置在$[p_{\min }, p_{\max }]$范围内的字符串。