---
title: 维吉尼亚（Vigenère）密码简介及破解方法
toc: true
date: 2018-04-03 11:26:43
tags:
mathjax: true
urlname: vigenere-cipher-and-cracking
---

在多表代换密码体制中，一个字母可以被映射为m个字母中的某一个（假定密钥字包含m个不同的字母）。维吉尼亚（Vigenère，又译维热纳尔）密码是一种典型的多表代换密码。一般来说，多表代换密码比单表代换密码更为安全一些，不过仍然能够通过各种分析手段进行破解。

## 加密和解密方法
首先选定一个任意的密钥$K = (k\_1, k\_2, ..., k\_m)$，将明文串转化为相应的数字，每m个为一组，使用密钥进行模26下的加密运算。

$$e\_k(x\_1, x\_2, ..., x\_m) = (x\_1 + k\_1, x\_2 + k\_2, ..., x\_m + k\_m)$$

$$d\_k(y\_1, y\_2, ..., y\_m) = (y\_1 - k\_1, y\_2 - k\_2, ..., y\_m - k\_m)$$

下面给出一个加密的具体例子。

### 例1
若m=6，密钥为`CIPHER`，要加密的明文为`thiscryptosystemisnotsecure`。<a href="#note1" id="note1ref"><sup>[1]</sup></a>

将明文串转化为对应的数字，使用密钥进行加密，如下所示：

| - | 0 | 1 | 2 | 3 | 4 | 5 |
| --- | --- | --- | --- | --- | --- | --- |
| **明文** | T(19) | H(7) | I(8) | S(18) | C(2) | R(17) |
| 密钥 | C(2) | I(8) | P(15) | H(7) | E(4) | R(17) |
| 密文 | V(21) | P(15) | X(23) | Z(25) | G(6) | I(8) |
| **明文** | Y(24) | P(15) | T(19) | O(14) | S(18) | Y(24) |
| 密钥 | C(2) | I(8) | P(15) | H(7) | E(4) | R(17) |
| 密文 | A(0) | X(23) | I(8) | V(21) | W(22) | P(15) |
| **明文** | S(18) | T(19) | E(4) | M(12) | I(8) | S(18) |
| 密钥 | C(2) | I(8) | P(15) | H(7) | E(4) | R(17) |
| 密文 | U(20) | B(1) | T(19) | T(19) | M(12) | J(9) |
| **明文** | N(13) | O(14) | T(19) | S(18) | E(4) | C(2) |
| 密钥 | C(2) | I(8) | P(15) | H(7) | E(4) | R(17) |
| 密文 | P(15) | W(22) | I(8) | Z(25) | I(8) | T(19) |
| **明文** | U(20) | R(17) | E(4) | - | - | - |
| 密钥 | C(2) | I(8) | P(15) | - |- | - |
| 密文 | W(22) | Z(25) | T(19) | - | - | - |

因此，得到密文为`VPXZGIAXIVWPUBTTMJPWIZITWZT`。

解密过程和加密过程恰好相反，不再赘述。

### 破解方法
首先给出英文字母出现的一般概率。虽然这一概率不能直接用于分析维吉尼亚密码的密文，但维吉尼亚密码并未完全抹消明文中的统计特性。

| 字母 | 概率 | 字母 | 概率 |
|:---:|:---:|:---:|:---:|
| A | 0.082 | N | 0.067 |
| B | 0.015 | O | 0.075 |
| C | 0.028 | P | 0.019 |
| D | 0.043 | Q | 0.001 |
| E | 0.127 | R | 0.060 |
| F | 0.022 | S | 0.063 |
| G | 0.020 | T | 0.091 |
| H | 0.061 | U | 0.028 |
| I | 0.070 | V | 0.010 |
| J | 0.002 | W | 0.023 |
| K | 0.008 | X | 0.001 |
| L | 0.040 | Y | 0.020 |
| M | 0.024 | Z | 0.001 |

#### 用Kasiski测试确定密钥长度m
Kasiski测试基于这一特性：如果明文中有两个相同的明文串，它们的位置间距为$\delta$，且$\delta \pmod 0 (mod m)$，则显然这两个明文串将被加密成相同的密文串。由于正常的英文明文中应该有很多词是重复出现的（如the，and等），我们可以利用这一特点，在密文中寻找相同的长度至少为3的密文串，它们很可能对应了相同的明文串，且位置间距为m的倍数。

事实上，我们可以进行一种一般化的称为“叠印”（superposition）的分析：
1. 将密文复制一份，固定其中一份的位置，另一份与之对齐
2. 将另一份密文左移一个字母
3. 计算两份密文此时相同的字母个数。当两份密文的位置间距为m的倍数时，重合个数会大幅度增加，原因是此时相邻的字母使用的是同一字母表（虽然是经过平移的）
4. 如果还没有找到可靠的m，转2

下面举两个例子进行具体说明。

##### 例2
已知使用维吉尼亚密码加密获得以下密文：
```
CHREEVOAHMAERATBIAXXWTNXBEEOPHBSBQMQEQERBW
RVXUOAKXAOSXXWEAHBWGJMMQMNKGRFVGXWTRZXWIAK
LXFPSKAUTEMNDCMGTSXMXBTUIADNGMGPSRELXNJELX
VRVPRTULHDNQWTWDTYGBPHXTFALJHASVBFXNGLLCHR
ZBWELEKMSJIKNBHWRJGNMGJSGLXFEYPHAGNRBIEQJT
AMRVLCRREMNDGLXRRIMGNSNRWCHRQHAEYEVTAQEBBI
PEEWEVKAKOEWADREMXMTBJJCHRTKDNVRZCHRCLQOHP
WQAIIWXNRMGWOIIFKEE
```
请尝试进行破解。<a href="#note2" id="note2ref"><sup>[2]</sup></a>

可以发现，密文中重复次数最多的串是`CHR`，出现了5次，且每次出现之间的间距分别为165、235、275、285，它们的最大公约数为5，因此我们猜测密钥的长度m=5。再进行叠印分析，得到以下数据：

| 偏移量 | 重合字母个数 |
|:---:|:---:|
| 1 | 14 |
| 2 | 15 |
| 3 | 12 |
| 4 | 11 |
| **5** | **22** |
| 6 | 14 |
| 7 | 13 |
| 8 | 15 |
| 9 | 15 |

可以发现，偏移量为5时相同字母个数最多，猜测密钥的长度m=5，与上述分析相符。

##### 例3
已知使用维吉尼亚密码加密获得以下密文：
```
KCCPKBGUFDPHQTYAVINRRTMVGRKDNBVFDETDGILTXR
GUDDKOTFMBPVGEGLTGCKQRACQCWDNAWCRXIZAKFTLE
WRPTYCQKYVXCHKFTPONCQQRHJVAJUWETMCMSPKQDYH
JVDAHCTRLSVSKCGCZQQDZXGSFRLSWCWSJTBHAFSIAS
PRJAHKJRJUMVGKMITZHFPDISPZLVLGWTFPLKKEBDPG
CEBSHCTJRWXBAFSPEZQNRWXCVYCGAONWDDKACKAWBB
IKFTIOVKCGGHJVLNHIFFSQESVYCLACNVRWBBIREPBB
VFEXOSCDYGZWPFDTKFQIYCWHJVLNHIQIBTKHJVNPIS
T
```
请尝试进行破解。<a href="#note3" id="note3ref"><sup>[3]</sup></a>

可以发现，密文中重复次数最多的串是`HJV`，出现了5次，且每次出现之间的间距分别为18、156、210、222，它们的最大公约数为6，因此我们猜测密钥的长度m=6。再进行叠印分析，得到以下数据：
Offset=1, cnt=12
Offset=2, cnt=9
Offset=3, cnt=9
Offset=4, cnt=13
Offset=5, cnt=13
Offset=6, cnt=16
Offset=7, cnt=16
Offset=8, cnt=11
Offset=9, cnt=10
| 偏移量 | 重合字母个数 |
|:---:|:---:|
| 1 | 12 |
| 2 | 9 |
| 3 | 9 |
| 4 | 13 |
| 5 | 13 |
| 6 | 16 |
| 7 | 16 |
| 8 | 11 |
| 9 | 10 |

可以发现，偏移量为6和7时相同字母个数最多，并不能立刻确定m是6还是7。这反映了叠印分析的弱点，当可供分析的密文比较多时，统计特征才会变得更明显。

## 参考文献
<a id="note1" href="#note1ref"><sup>[1]</sup></a>此题来自《密码学原理与实践》第二版例1.4。
<a id="note2" href="#note2ref"><sup>[2]</sup></a>此题来自《密码学原理与实践》第二版例1.12。
<a id="note3" href="#note3ref"><sup>[3]</sup></a>此题来自《密码学原理与实践》第二版练习1.21（b）。

[这个网站](https://www.dcode.fr/vigenere-cipher)提供了加密和各种解密工具。
