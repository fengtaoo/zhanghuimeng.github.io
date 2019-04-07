---
title: RSA密码体制
urlname: rsa-cryptosystem
toc: true
mathjax: true
date: 2018-06-05 19:25:37
tags: Cryptography
---

内容主要来自《密码学原理与实践》第5章和《现代密码学》中讲授的RSA部分。

## 公钥密码体制

## RSA密码体制

### 一些数学知识

了解RSA算法自身所需的数学知识并不多。

#### 扩展欧几里得算法

简单来说，这个算法是对欧几里得算法的扩展：对于两个正整数$a, b$，这个算法在求出$gcd(a, b)$的同时，还能求出相对应的整数$x, y$，满足$ax + by = gcd(a, b)$。

书上对扩展欧几里得算法的描述非常不直观。实际上，我觉得这个算法的思路是非常直接的，不需要非得用形式化的方法来表示。[维基](https://zh.wikipedia.org/wiki/%E6%89%A9%E5%B1%95%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%97%E7%AE%97%E6%B3%95)上的做法就非常直接。下面尝试用这种做法来处理书上的例5.1：

计算$28^{-1} \bmod 75$。

由于$gcd(28, 75) = 1$，只需令$a = 28, b = 75$，用扩展欧几里得算法求出对应的$x, y$使得$28x + 75y = 1$，由于$28x + 75y \equiv 28x \equiv 1 (\bmod 75)$，因此$x$即为28模75的乘法逆元。下面给出具体的求解过程：

* 首先用辗转相除法的形式写出欧几里得算法的过程：
  * $28 = 75 \times 0 + 28$
  * $75 = 28 \times 2 + 19$
  * $28 = 19 \times 1 + 9$
  * $19 = 9 \times 2 + 1$
* 将余数移到等号左边：
  * $28 = 28 - 75 \times 0$
  * $19 = 75 - 28 \times 2$
  * $9 = 28 - 19 \times 1$
  * $1 = 19 - 9 \times 2$
* 将每一步的余数代入到下面的式子中，不断消去余数，最后得到$gcd(a, b)$与$a, b$的关系：
  * $28 = 28 - 75 \times 0$
  * $19 = 75 - 28 \times 2 = 75 - (28 - 75 \times 0) \times 2 = 75 - 28 \times 2$
  * $9 = 28 - 19 \times 1 = 28 - (75 - 28 \times 2) \times 1 = -75 + 28 \times 3$
  * $1 = 19 - 9 \times 2 = (75 - 28 \times 2) - (-75 + 28 \times 3) \times 2 = 75 \times 3 - 28 \times 8$

解得$x = -8, y = 3$，因此$28^{-1} \equiv -8 \equiv 67 (\bmod 75)$

### 具体算法

RSA密码体制的形式定义如下：

设$n = pq$，其中p和q为素数。设$\mathcal{P} = \mathcal{Q} = \mathbb{Z}_n$，定义

$$\mathcal{K} = \{(n, p, q, a, b): ab \equiv 1\ (\bmod \phi(n))\}$$

对于$K = (n, p, q, a, b)$，定义

$$e_K(x) = x^b \bmod n$$

$$d_K(y) = y^a \bmod n$$

其中$x, y \in \mathbb{Z}_n$。值$n$和$b$组成了公钥，而值$p, q, a$组成了私钥。

下面证明加密和解密是逆运算。（主要参考了[阮一峰的证明](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)。）



## 素性检验（寻找大素数）

### Solovay-Strassen算法

#### 一些数学知识

#### 具体算法

### Miller-Rabin算法

```
Miller-Rabin(n)
    把n-1写成n-1 = 2^k * m，其中m是一个奇数
    选取随机整数a，使得1 <= a <= n-1
    b = a^m (mod n)
    if b == 1 (mod n)
        return ("n is prime")
    for i = 0 to k-1
        if b == -1 (mod n)
            return ("n is prime")
        else
            b = b^2 (mod n)
    return ("n is composite")
```

## 攻击方法

### 分解公开模数的攻击（分解因子算法）

### 解密指数攻击

### Wiener的低解密指数攻击
