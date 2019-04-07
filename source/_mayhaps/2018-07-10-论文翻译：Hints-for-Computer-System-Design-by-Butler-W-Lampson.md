---
title: '论文翻译："Hints for Computer System Design" by Butler W. Lampson（更新中）'
urlname: hints-for-computer-system-design-translation
toc: true
date: 2018-07-10 19:12:48
tags: [OS]
---

## 写在翻译之前

上回在[OSTEP第5章](/post/ostep-ch-05-summary-interlude-process-api/)中读到“Get it right”这个说法之后，我起初没当回事，之后却经常觉得这句话很有道理，特别是在翻译[介绍Github Spokes的那几篇文章](/tags/Spokes/)的时候。直接利用Git复制三份存储库作为备份——这个做法非常漂亮，但它是**正确**的吗？这样做是否会浪费过多的存储空间？当然这里我并不是要说这个做法不正确，只是在翻译的时候想到了这回事而已。后来，读到[Why Google Stores Billions of Lines of Code in a Single Repository](https://cacm.acm.org/magazines/2016/7/204032-why-google-stores-billions-of-lines-of-code-in-a-single-repository/fulltext)这篇文章的时候，这种感觉更强烈了：Git就是**正确**的事吗？到底在何种场景下，中心化的存储管理会更有优势？（当然，我现在对此的回答是，分布式存储是大势所趋；但事情应该也没有这么简单。）这些都使得我想要去找来作者的原文读一读。

稍微浏览过一下之后，我发现了两件有趣的事情：

1. 作者似乎十分喜欢《哈姆雷特》，全文中充满了来自《哈姆雷特》的引文。
2. 网上好像还没有中文译文。

哈，虽然我对那位忧郁拖延的王子没有太多兴趣，但我还是挺喜欢莎士比亚的，所以我就心血来潮，决定来翻译一下了！

最近我学到的一个道理是，尽快开始做，然后在不断迭代中修改，要比呆坐着沉思，考虑自己怎么才能做得更好要强上一万倍（这也是我对哈姆雷特殿下有偏见的原因……）。所以我会借助谷歌翻译先大概翻译一下，然后进行修改，之后尝试发布到别的公众平台（因为实在没有人读我这个独立博客），再继续进行修改。最后应该会得到若干个正式的PDF译文版本吧。

好的，开始吧。

原文地址：[Hints for Computer System Design](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/acrobat-17.pdf)

### 更新日志



---

## 摘要

<!--
Studying the design and implementation of a number of computer has led to some general hints for system design. They are described here and illustrated by many examples, ranging from hardware such as the Alto and the Dorado to application programs such as Bravo and Star.
-->

对一部分计算机的设计和实现的研究已经产生了一些用于系统设计的一般性的提示。本文将描述这些提示，并通过许多例子来说明它们，例子的范围包括Alto<a href="#note1" id="note1ref"><sup>1</sup></a>和Dorado等硬件，也包括Bravo和Star等应用程序。

## 1. 简介

<!--
Designing a computer system is very different from designing an algorithm:
The external interface (that is, the requirement) is less precisely defined, more complex, and more subject to change.
The system has much more internal structure, and hence many internal interfaces.
The measure of success is much less clear.
-->

设计计算机系统与设计算法有着很大的区别：

* 外部接口（即需求）的定义更不精确，更复杂，且更易变化。
* 系统具有更多的内部结构，因此有许多内部接口。
* 设计成功的标准更模糊。

<!--
The designer usually finds himself floundering in a sea of possibilities, unclear about how one choice will limit his freedom to make other choices, or affect the size and performance of the entire system. There probably isn’t a ‘best’ way to build the system, or even any major part of it; much more important is to avoid choosing a terrible way, and to have clear division of responsibilities among the parts.
-->

设计者经常会发现自己在可能性的海洋中挣扎，不清楚做出某一选择将如何限制他做出其他选择的自由，并影响整个系统的规模和性能。可能没有一种“最佳”的方式来构建系统，即便只是它的某一主要部分；更重要的是避免选择一种糟糕的方式，并明确划分各部分之间的责任。

<!--
I have designed and built a number of computer systems, some that worked and some that didn’t. I have also used and studied many other systems, both successful and unsuccessful. From this experience come some general hints for designing successful systems. I claim no originality for them; most are part of the folk wisdom of experienced designers. Nonetheless, even the expert often forgets, and after the second system [6] comes the fourth one.
-->

我设计并构建过许多计算机系统，其中一些能够工作，有些则不能。我还使用和研究过许多其他系统，有些是成功的，有些则不成功。从这些经验中产生了一些如何设计成功的系统的一般性的提示。 我不能自称为这些想法的原创者；其中的大多数来自经验丰富的设计师的集体智慧。然而，专家也会经常忘记这些事情，在第二个系统之后设计的就是第四个系统<a href="#bib6" id="bib6ref"><sup>[6]</sup></a>。

<!--
Disclaimer: These are not
novel (with a few exceptions),
foolproof recipes,
laws of system design or operation,
precisely formulated,
consistent,
always appropriate,
approved by all the leading experts, or
guaranteed to work.

They are just hints. Some are quite general and vague; others are specific techniques which are more widely applicable than many people know. Both the hints and the illustrative examples are necessarily oversimplified. Many are controversial.
-->

免责声明：这些内容不是
小说（除了个别例外）、
万无一失的策略、
系统设计和运行的律法，
一致、
精确、
永远恰当的，
未受全部专家承认，
也不保证有效，

它们只是提示。有些提示是相当笼统和含糊的；另一些则是可以在更广的范围内应用的特定技术。这些提示和用于说明的示例都经过了必要的过度简化。很多提示是有争议的。

<!--
I have tried to avoid exhortations to modularity, methodologies for top-down, bottom-up, or iterative design, techniques for data abstraction, and other schemes that have already been widely disseminated. Sometimes I have pointed out pitfalls in the reckless application of popular methods for system design.
-->

## 我的注解

<a id="note1" href="#note1ref"><sup>1</sup></a>


## 参考文献

<a id="bib1" href="#bib1ref"><sup>[1]</sup></a>

<a id="bib6" href="#bib6ref"><sup>[6]</sup></a>Brooks, F.H. The Mythical Man-Month, Addison-Wesley, 1975.
我在软件工程课上听说过这本书，但是没有读过，不知道具体说的是什么。
