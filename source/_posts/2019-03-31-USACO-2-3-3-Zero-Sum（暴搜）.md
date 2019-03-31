---
title: 'USACO 2.3.3: Zero Sum（暴搜）'
urlname: usaco-2-3-3-zero-sum
toc: true
date: 2019-03-31 16:39:18
updated: 2019-03-31 16:39:18
tags: [USACO, alg:Brute Force]
categories: USACO
---

## 题意

见[洛谷 P1473 零的数列 Zero Sum](https://www.luogu.org/problemnew/show/P1473)。

对于数列`1 2 3 ... N`（`3 <= N <= 9`），在除了1之外的每个数前面插入` `、`+`或`-`，如果得到的和为0，就把这个数列输出。

## 分析

这道题明摆着是要直接搜索。搜索空间总共只有`3^8 = 6561`，所以应该还挺快的。

## 代码

```cpp
/*
ID: zhanghu15
TASK: zerosum
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>

using namespace std;
typedef long long int LL;

ofstream fout("zerosum.out");
ifstream fin("zerosum.in");

int N;
char signs[10];

void dfs(int depth) {
    if (depth == N - 1) {
        LL sum = 0, cur = 1;
        int s = 1;  // sign
        for (int i = 0; i < depth; i++) {
            if (signs[i] == ' ') cur = cur * 10 + (i + 2);
            else if (signs[i] == '+') {
                sum += cur * s;
                s = 1;
                cur = i + 2;
            }
            else {
                sum += cur * s;
                s = -1;
                cur = i + 2;
            }
        }
        sum += cur * s;
        if (sum == 0) {
            fout << 1;
            for (int i = 0; i < depth; i++)
                fout << signs[i] << (i + 2);
            fout << endl;
        }
        return;
    }
    signs[depth] = ' ';
    dfs(depth + 1);
    signs[depth] = '+';
    dfs(depth + 1);
    signs[depth] = '-';
    dfs(depth + 1);
}

int main() {
    fin >> N;
    dfs(0);
    return 0;
}
```
