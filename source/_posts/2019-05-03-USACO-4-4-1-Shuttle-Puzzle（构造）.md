---
title: 'USACO 4.4.1: Shuttle Puzzle（构造）'
urlname: usaco-4-4-1-shuttle-puzzle
toc: true
date: 2019-05-03 02:09:42
updated: 2019-05-03 15:48:00
tags: [USACO, alg:Ad-Hoc]
categories: USACO
---

这道题使用的构造法还是挺有意思的。

<!--more-->

## 题意

见[洛谷 P2739 棋盘游戏 Shuttle Puzzle](https://www.luogu.org/problemnew/show/P2739)。

给定一个有`2N+1`个孔的棋盘，上面放着`N`个白色棋子和`N`个黑色棋子，初始状态为

```txt
WW..W_B..BB
```

目标状态为

```txt
BB..B_W..WW
```

只能进行两种操作：

* 将空孔一侧的棋子移入孔中
* 将棋子跳过另**一枚**棋子，进入空孔中

用每次移动的棋子在棋盘上的位置表示一次移动，求出移动次数最小且字典序最小的移动步骤。

## 分析

这道题看起来像是搜索（实际上，用搜索确实也能做），实际上应该用构造法。题目里已经给了`N=3`时的正解：

<!-- ↷ ↶ → ← -->

```txt
WWW BBB  3 W→
WW WBBB  5 B↶
WWBW BB  6 B←
WWBWB B  4 W↷
WWB BWB  2 W↷
W BWBWB  1 W→
 WBWBWB  3 B↶
BW WBWB  5 B↶
BWBW WB  7 B↶
BWBWBW   6 W→
BWBWB W  4 W↷
BWB BWW  2 W↷
B BWBWW  3 B←
BB WBWW  5 B↶
BBBW WW  4 W→
BBB WWW
```

显然可以看出棋盘的变化是前后对称的，而且总移动步数是`(N + 1)^2`。

我自己从中（经过多次试错和在纸上的模拟）总结出的规律是：

* 按照`W→`，`B↶`，`B←`，`W↷`的顺序进行移动
* 右移W时，如果是总移动步数前半部分的移动，只在空格右侧不是W时移动；左移B时同理
* 向右跳W时，不跳过W；左跳B时同理

然后直接模拟就可以了。

不过，显然还有更好的方法。如果盯着这个棋盘变化表多看一会儿，会发现空格的位置是有规律的，好像形成了几条斜线（下面两张图摘自USACO题解）：

![移动棋子的位置](prob27.gif)

![把上述位置连起来](prob27a.gif)

空格位置的变化和每一步移动的棋子的位置实际上是相同的（`N = 3`，把第一步也算上）：

```txt
4 3 5 6 4 2 1 3 5 7 6 4 2 3 5 4
```

可以看出，上述数列是若干个等差数列的组合：

```txt
4 ; 3 5 ; 6 4 2 ; 1 3 5 7 ; 6 4 2 ; 3 5 ; 4
```

根据类似的规律，也可以进行求解。

## 代码

```cpp
/*
ID: zhanghu15
TASK: shuttle
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>
#include <map>
#include <queue>

using namespace std;
typedef long long int LL;

ofstream fout("shuttle.out");
ifstream fin("shuttle.in");

char state[200][30];
int space[200];
int moves[200];

int N, M, m;

int main() {
    fin >> N;
    for (int i = 0; i < N; i++)
        state[0][i] = 'W';
    state[0][N] = ' ';
    for (int i = N + 1; i <= 2*N; i++)
        state[0][i] = 'B';
    space[0] = N;

    M = (N + 1) * (N + 1);
    int flag = 0; // 使用了类似于状态机的思路
    for (m = 0; m < M - 1; m++) {
        memcpy(state[m + 1], state[m], sizeof(state[m]));
        while (true) {
            // move W right
            if (flag == 0) {
                if (space[m] > 0 && state[m][space[m] - 1] == 'W' &&
                    (m >= M/2 || space[m] == 2*N || state[m][space[m] + 1] != 'W')) {
                    swap(state[m + 1][space[m]], state[m + 1][space[m] - 1]);
                    space[m + 1] = space[m] - 1;
                    moves[m] = space[m] - 1;
                    break;
                }
                flag = 1;
            }
            // jump B left
            if (flag == 1) {
                if (space[m] + 2 <= 2*N && state[m][space[m] + 2] == 'B' && state[m][space[m] + 1] != 'B') {
                    swap(state[m + 1][space[m]], state[m + 1][space[m] + 2]);
                    space[m + 1] = space[m] + 2;
                    moves[m] = space[m] + 2;
                    break;
                }
                flag = 2;
            }
            // move B left
            if (flag == 2) {
                if (space[m] + 1 <= 2*N && state[m][space[m] + 1] == 'B' &&
                    (m >= M/2 || space[m] == 0 || state[m][space[m] - 1] != 'B')) {
                    swap(state[m + 1][space[m]], state[m + 1][space[m] + 1]);
                    space[m + 1] = space[m] + 1;
                    moves[m] = space[m] + 1;
                    break;
                }
                flag = 3;
            }
            // jump W right
            if (flag == 3) {
                if (space[m] - 2 >= 0 && state[m][space[m] - 2] == 'W' && state[m][space[m] - 1] != 'W') {
                    swap(state[m + 1][space[m]], state[m + 1][space[m] - 2]);
                    space[m + 1] = space[m] - 2;
                    moves[m] = space[m] - 2;
                    break;
                }
                flag = 0;
            }
        }
        cout << state[m + 1] << endl;
    }
    for (int i = 0; i < M - 1; i++) {
        if (i % 20 != 0) fout << ' ';
        if (i % 20 == 0 && i != 0) fout << endl;
        fout << moves[i] + 1;
    }
    fout << endl;
    return 0;
}
```
