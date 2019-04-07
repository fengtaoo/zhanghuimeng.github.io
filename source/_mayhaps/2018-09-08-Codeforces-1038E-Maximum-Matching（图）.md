---
title: Codeforces 1038E. Maximum Matching（图）
urlname: codeforces-1038e-maximum-matching
toc: true
date: 2018-09-08 17:04:40
updated: 2018-09-08 17:04:40
tags: [Codeforces, Codeforces Contest, alg:Graph, alg:Brute Force, alg:Depth-first Search]
---

题目来源：[http://codeforces.com/contest/1038/problem/E](http://codeforces.com/contest/1038/problem/E)

提交次数：1/9

## 题意

给定`n`个方块，每个方块形如`[color_1 | value | color_2]`（`1 <= color <= 4`，`1 <= value <= 100000`），且方块可以左右翻转。要求从这些方块里挑出一些排成一行，且相邻的方块的相邻面的颜色相等。问所有方块的`value`之和的最大值是多少。

## 分析

我之前好像说过……题的形式之前曾经见过（[SGU 101](https://codeforces.com/problemsets/acmsguru/problem/99999/101)，当时我觉得模型的抽象方法是一样的：也就是说，以`color`作为结点，方块作为结点之间连接的边。但是这道题不保证有欧拉路。那么怎么求最大值呢？我思考了一下。很显然，如果访问到一个结点，那么就相当于访问过这个结点的所有自环了。所以我们可以暂时不考虑这些自环，把它们移除掉。以及，如果两个结点之间有`2k + m`条边（`0 <= m <= 1`），则`2k`条边没有意义，因为我们可以在这两个结点之间往返`k`次，访问这些边，然后回到出发时的结点。此时，我们考虑的问题就变成了，如何移除没有被访问到的边？

这样我们就可以写出一个这样的算法：首先建一张只有4个结点的图（4种颜色）的邻接矩阵，然后把所有方块的`value`插入到对应的邻接矩阵中；然后计算得到每两个结点之间的边数模2的值。根据这个值重建一张新图，用DFS的方法在其中寻找所有可能的路径。然后根据路径的访问情况判断每两个结点之间的边能否被全部访问；如果不能被全部访问，则删除其中`value`最小的边。
