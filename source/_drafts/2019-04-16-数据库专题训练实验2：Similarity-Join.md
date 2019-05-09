---
title: 数据库专题训练实验2：Similarity Join
urlname: advanced-database-exp-2-similarity-join
toc: true
mathjax: true
date: 2019-04-16 16:28:01
updated: 2019-04-16 16:28:01
tags:
categories:
---

这是数据库专题训练的第2个大作业，实现Similarity Join。

<!--more-->

## 时间测试框架

我决定采用论文中使用的两个数据集进行测试：

* Edit Distance：[DBLP](http://dbgroup.cs.tsinghua.edu.cn/ligl/simjoin/data/dblp.tar.bz2)
* Jaccard：[Enron](http://dbgroup.cs.tsinghua.edu.cn/ligl/simjoin/data/enron.tar.bz2)

## Pass-Join

这个算法的基本框架也是filter-and-refine：先筛掉很多不可能的对，再遍历所有剩下的对，进行精确计算。

算法的主要原理可以通过下列引理说明：将$s$分成$\tau + 1$段，记ed要求的阈值为$\tau$，根据鸽巢原理，如果$s$和$r$在阈值$\tau$的范围内相似，则$r$中必然有一个子串与$s$的一段匹配。本算法中进行的是均分，即后$k=|s|-\left\lfloor\frac{|s|}{\tau+1}\right\rfloor * (\tau + 1)$段长度为$\left\lceil\frac{|s|}{\tau+1}\right\rceil$，前$\tau + 1 - k$段长度为$\left\lfloor\frac{|s|}{\tau+1}\right\rfloor$。

显然，如果$r$中完全没有匹配$s$的某一段的子串，$s$和$r$不可能相似，所以可以进行剪枝。下面描述剪枝的算法。

一些记号：

* $\mathcal{S}_l$：长度为$l$的字符串集合
* $\mathcal{S}_l^i$：$\mathcal{S}_l$中每个字符串第$i$小段组成的集合
* $\mathcal{L}_{l}^i$：$\mathcal{S}_l^i$的倒排列表
* $\mathcal{L}_l^i(\omega)$：字符串$\omega$在$\mathcal{L}_l^i$中的查询结果，即$\mathcal{S}_l$中第$i$个小段为$\omega$的字符串集合

PASS-JOIN主算法流程（self-join）：

1. 对所有字符串按长度和字典序排序（长度优先）
2. 遍历字符串，对于当前字符串$s$，基于length filtering，在一些倒排列表$\mathcal{L}_l^i \, (|s|-\tau \leq l \leq|s|, 1 \leq i \leq \tau+1)$中查找相似的字符串（这些倒排列表只包含已经访问过的字符串，这是为了防止重复）
3. 不失一般性，考虑如何在$\mathcal{L}_l^i$中查找相似的字符串
   * 执行SUBSTRING SELECTION算法流程：从$s$中选出一些子串（显然naive方法是遍历$s$的所有子串），记为$\mathcal{W}\left(s, \mathcal{L}_l^i\right)$，对于$\forall w \in \mathcal{W}\left(s, \mathcal{L}_l^i\right)$，如果$\omega$出现在$\mathcal{L}_l^i$中，则对于$\forall r$，$r \in \mathcal{L}_l^i(w)$，$\langle r, s\rangle$可作为候选对
   * 执行VERIFICATION算法流程：对每个候选对$\langle r, s\rangle$，计算两者的编辑距离
4. 将$s$分成$\tau + 1$个小段，插入到对应的倒排列表$\mathcal{L}_{|s|}^i \quad (1 \leq i \leq \tau+1)$中。（因为$s$的长度是递增的，所以此时可以删除满足$k < |s| - \tau$的倒排列表$\mathcal{L}_k^i$。）

PASS-JOIN主算法流程（$\mathcal{R}$和$\mathcal{S}$）：

1. 将$\mathcal{R}$和$\mathcal{S}$中的字符串分别按长度和字典序排序
2. 将$\mathcal{S}$中的全部字符串按照上述方法创建倒排列表
3. 遍历$\mathcal{R}$中的字符串
   * 对于长度为$|r|$的字符串，在$[|r|-\tau,|r|+\tau]$长度范围内对应的倒排列表中查找相似的字符串
   * 删除长度小于$|r| - \tau$的字符串对应的倒排列表

现实是，如果直接按照上述方法写，内存会爆掉，所以不能直接把$\mathcal{S}$中的全部字符串都建成倒排列表，也需要跟着遍历$\mathcal{R}$时字符串长度变化的过程新建和删除。

### SUBSTRING SELECTION的改进

论文里这部分写的好像只针对self-join的情况。。。

记$\mathcal{W}(s, l)=\cup_{i=1}^{\tau+1} \mathcal{W}\left(s, \mathcal{L}_l^i\right)$（$|s| - \tau \leq l \leq |s|$），我们只使用这些子串进行SUBSTRING SELECTION。（如果不是self-join，应该是$|s| - \tau \leq l \leq |s| + \tau$。）

最简单的方法是把$s$的所有子串都放到$\mathcal{W}(s, l)$中，则子串总数为$\sum_{i=1}^{|s|}(|s|-i+1)=\frac{|s| *(|s|+1)}{2}$，比较昂贵。

下面着重于优化$\mathcal{W}\left(s, \mathcal{L}_l^i\right)$。

#### 单纯基于长度的方法

显然$\mathcal{L}_l^i$中索引的小段长度都相同（不妨记为$l_i$），显然可以只选择$s$中长度为$l_i$的子串。记这种选择方法为$\mathcal{W}_l\left(s, \mathcal{L}_l^i\right)$。

#### 基于长度和位移的方法

$\mathcal{L}_l^i$中索引的小段除了长度相同之外，在原字符串中的起始位置也相同（不妨记为$p_i$），我们可以只选择$s$中起始位置在$\left[p_i-\tau, p_i+\tau\right]$内且长度为$l_i$的子串。记这种选择方法为$\mathcal{W}_f\left(s, \mathcal{L}_l^i\right)$。

这种方法的主要原理是忽略了一些重复。

#### 基于位移和长度差的方法

{% raw %}
考虑$\mathcal{L}_l^i$中与$s$匹配的字符串$r$，假设它在$p_i$处长度为$l_i$的小段与$s$在$p$处长度相同的小段匹配。记$s(r)$在匹配处左边的子串为$s_l(r_l)$，匹配处的子串为$s_m(r_m)$，右边的子串为$s_r(r_r)$。假定$s$和$r$在$s_m = r_m$处对齐，记$d_{l}=\mathrm{ED}\left(r_l, s_l\right)$，$d_r=\mathrm{ED}\left(r_r, s_r\right)$，为使$s$和$r$相似，必有$d_l+d_r \leq \tau$。
{% endraw %}

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

{% raw %}
如果$\Delta_r \leq \Delta$，则$d_r \geq \Delta - \Delta_r$，因此$\Delta = \Delta_r + (\Delta - \Delta_r) \leq d_l + d_r \leq \tau$；否则$\Delta_r > \Delta$，$d_r \geq \Delta_r - \Delta$，$\tau \geq d_l + d_r \geq \Delta_r + (\Delta_r - \Delta)$，即$\Delta_r \leq\left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$。在$|s| - \tau \leq l \leq |s| + \tau$的前提下（这是length filter），显然有$\Delta \leq \tau$，$\Delta \leq \left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$，因此这两种情况可以统一为$\Delta_r \leq \left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$，即$p=p_{i}+\Delta_r \leq p_{i}+\left\lfloor\frac{\tau+\Delta}{2}\right\rfloor$。
{% endraw %}

因此有

{% raw %}
$$
\begin{aligned}
p_{\min } &= \max \left(1, p_{i}-\left\lfloor\frac{\tau-\Delta}{2}\right\rfloor\right) \\
p_{\max } &= \min \left(|\mathrm{s}|-l_{i}+1, p_{i}+ \left\lfloor \frac{\tau+\Delta}{2} \right\rfloor \right)
\end{aligned}
$$
{% endraw %}

---

在$|s| < l$的情况下，记$\Delta = |r| - |s|$，$\Delta_l = p_i - p$，$\Delta_r = p - p_i$。

当$p \leq p_i$时，如下图：

![p <= pi](position-aware-min-alt.jpg)

$$d_l \geq \Delta_l$$

$$d_r \geq |\Delta_l - \Delta|$$

类似地，可以得到

$$p \geq p_i - \left\lfloor\frac{\tau + \Delta}{2}\right\rfloor$$

当$p > p_i$时，如下图：

![p > pi](position-aware-max-alt.jpg)

$$d_l \geq \Delta_r$$

$$d_r \geq \Delta_r + \Delta$$

类似地，可以得到

$$p \leq p_{i}+\left\lfloor\frac{\tau - \Delta}{2}\right\rfloor$$

因此有

{% raw %}
$$
\begin{aligned}
p_{\min } &= \max \left(1, p_{i}-\left\lfloor\frac{\tau + \Delta}{2}\right\rfloor\right) \\
p_{\max } &= \min \left(|\mathrm{s}|-l_{i}+1, p_{i}+ \left\lfloor \frac{\tau - \Delta}{2} \right\rfloor \right)
\end{aligned}
$$
{% endraw %}

考虑到此处$\Delta$的定义，如果将$\Delta$修改为$\Delta = |s| - |r|$，则上述表达式的形式应该与$l \leq |s|$的情形相同。

结论：我们可以只选择长度为$l_i$且起始位置在$[p_{\min }, p_{\max }]$范围内的字符串。

#### 基于多次匹配的方法

待写。。。

### VERIFICATION的改进

#### 动态规划

显然可以直接

## PP-Join

同时进行Prefix Filtering和Positiona; Filtering，然后进行Verification。

首先把Jaccard阈值换算成Overlap阈值。由定义，

$$O(x, y)=|x \cap y|$$

$$J(x, y)=\frac{|x \cap y|}{|x \cup y|}=\frac{O(x, y)}{|x|+|y|-O(x, y)}$$

因为$J(x, y) \geq t$，因此

$$\begin{aligned} \frac{O(x, y)}{|x|+|y|-O(x, y)} \geq t & \Longleftrightarrow(1+t) O(x, y) \geq t(|x|+|y|) \\
& \Longleftrightarrow O(x, y) \geq \frac{t}{1+t} \cdot(|x|+|y|) \end{aligned}$$

因此

$$J(x, y) \geq t \Longleftrightarrow O(x, y) \geq \alpha=\frac{t}{1+t} \cdot(|x|+|y|)$$

### Prefix + Positional Filtering

![ppjoin算法](ppjoin-filter.png)

一些定义：

* $I_w$：$w$对应的倒排列表
* $\alpha$：换算出的Overlap阈值
* $\operatorname{ubound}$
* $A[y]$：$x$和$y$在token $w$左侧的重合的token的数量

算法过程如下：

1. 初始化结果集合$S$
2. 初始化倒排列表$I$
3. 遍历$R$中的每个记录$x$
4. 初始化HashMap $A$
5. 计算前缀长度$p = |x|-\lceil t \cdot|x|\rceil+ 1$
6. 对于$x$的前$p$个token
7. 记$w = x[i]$
8. 对于$w$的倒排列表$I_w$中的每条满足$|y| \geq t \cdot|x|$的记录$y$（其中$w$是$y$的第$j$个token）
9. 计算$\alpha \leftarrow\left\lceil\frac{t}{1+t}(|x|+|y|)\right\rceil$
10. 计算$\operatorname{ubound} = 1+\min (|x|-i,|y|-j)$
11. 如果$A[y]+\operatorname{ubound} \geq \alpha$（即左边重合的token数量+右边最多可能重合的token数量>=换算出的overlap阈值）
12. 则$A[y] = A[y] + 1$（把token $w$加进去）
13. 否则
14. $A[y] = 0$（这一步相当于不直接的剪枝；因为Verify过程用到了$A$的信息，所以并不会做直接删除）
15. 将$x$和$i$插入到$I_w$中
16. 调用Verify过程
17. 返回$S$

### 优化过的Verify

![verify算法](ppjoin-verify.png)

因为已经知道前缀中重合的token数量了，所以可以进行各种优化。

一些定义：

* $p_x$：$x$的前缀长度
* $p_y$：$y$的前缀长度

算法过程：

1. 对于每个满足$A[y] > 0$的$y$（此处剪去了$A[y] = 0$的$y$）
2. 令$w_x$为$x$的前缀中的最后一个token
3. 令$w_y$为$y$的前缀中的最后一个token
4. 令$O = A[y]$
5. 如果$w_x < w_y$
6. $\operatorname{ubound} = A[y] + |x| - p_x$
7. ……
8. ……
9. ……
10. ……
11. ……
12. ………
13. ……

虽然不太理解但是还是直接写好了。

## Adapt-Join

这个方法可以处理Jaccard Similarity。

也是包括Prefix Filtering和Verification两个步骤。

### Prefix Filtering框架

首先把对象映射成集合。（文中要求不去重，但作业要求去重）然后把相似函数的阈值$\theta$换算成overlap threshold $t$：

$$\operatorname{sim}_j(r, s)=\frac{|r \cap s|}{|r|+|s|-|r \cap s|}$$

$$\operatorname{sim}_j(r, s) \geq \theta \Rightarrow |r \cap s| \geq \theta \cdot|r| \Rightarrow t=\lceil\theta \cdot|r|\rceil$$

（ed那些就不管了）

显然可以把交集大小小于$t$的对去掉。首先定义一个全序，然后将每个集合中的元素排序。记$Prefix(r)$是集合$r$的前$|r| - t + 1$个元素的集合，容易证明，如果$|r \cap s| \geq t$，则$Prefix\left(r_{1}\right) \cap Prefix\left(s_{1}\right) \neq \phi$。

然后就可以通过建立倒排列表的方法来进行筛选。