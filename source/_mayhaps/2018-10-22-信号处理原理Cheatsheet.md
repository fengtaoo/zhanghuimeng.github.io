---
title: 信号处理原理Cheatsheet
urlname: signal-processing-cheatsheet
toc: true
mathjax: true
date: 2018-10-22 23:21:59
updated: 2018-10-22 23:21:59
tags:
---

## 采样定理

冲激串采样（理想采样）：

$$p(t) = \sum_{n = -\infty}^{\infty} \delta(t - nT)$$

$$P(\omega) = \frac{2\pi}{T} \sum_{n = -\infty}^{\infty} \delta(\omega - \frac{2\pi}{T} n)$$

$$x_p(t) = x(t) p(t) \Leftrightarrow \frac{1}{2\pi} X(\omega) * P(\omega) = \frac{1}{T} \sum_{n=-\infty}^{\infty} X(\omega - n\omega_s)$$

结论：在时域对连续时间信号进行理想采样，相当于在频域对连续时间信号频谱以$\omega_s$为周期进行延拓。

结论：……不能发生频谱的混叠。因此要求：

1. $x(t)$是带限信号
2. $\omega_s \geq 2\omega_M$

## DTFT
