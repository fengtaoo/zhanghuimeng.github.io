---
title: Leetcode Weekly Contest 128总结
urlname: leetcode-weekly-contest-128
toc: true
date: 2019-03-31 15:37:36
updated: 2019-03-31 15:37:36
tags: [Leetcode, Leetcode Contest, alg:Binary Search]
categories: Leetcode
---

这周比赛的时候我睡过头了，所以并没有做完所有的题。最后一道题还是有点难度的！虽然我见过和它很像的题，可是还是没能立刻想出来。

<!--more-->

## [1012. Complement of Base 10 Integer](https://leetcode.com/problems/complement-of-base-10-integer/description/)

标记难度：Easy

提交次数：1/1

代码效率：100.00%（4ms）

### 题意

求十进制数的补码在十进制下的表示。补码不包括前导0（0除外）。

### 分析

所以直接`~N`显然是不行的（因为有很多个前导0）。

最简单的方法是直接把二进制表示手算出来，然后再手算补码。

另一种比较聪明的方法是，考虑到`N + N补 = 111...1`，且`N补`不为0，所以只需找到比`N`大的最小的二进制全为1的数，然后用它减去`N`即可。[^lee]

[^lee]: [lee215's Solution for Leetcode 1012 - \[Java/C++/Python\] Find 111.....1111 >= N](https://leetcode.com/problems/complement-of-base-10-integer/discuss/256740/JavaC++Python-Find-111.....1111-greater-N)

### 代码

```cpp
class Solution {
public:
    int bitwiseComplement(int N) {
        if (N == 0) return 1;
        int b = 1, ans = 0;
        while (N > 0) {
            if (!(N & 1)) ans |= b;
            N >>= 1;
            b <<= 1;
        }
        return ans;
    }
};
```

## [1013. Pairs of Songs With Total Durations Divisible by 60](https://leetcode.com/problems/pairs-of-songs-with-total-durations-divisible-by-60/description/)

标记难度：Easy

提交次数：1/1

代码效率：16.04%（60ms）

### 题意

求所有长度之和模60余0且满足`i < j`的`(i, j)`对。

### 分析

水题。

用一个`map`统计已经出现的长度模60的余数，然后对于当前长度，计算和它相加模60为0的余数个数即可。

### 代码

```cpp
class Solution {
public:
    int numPairsDivisibleBy60(vector<int>& time) {
        map<int, int> modSet;
        int ans = 0;
        for (int t: time) {
            int m = t % 60;
            ans += modSet[(60 - m) % 60];
            modSet[m]++;
        }
        return ans;
    }
};
```

## [1011. Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/description/)

标记难度：Medium

提交次数：1/1

代码效率：33.79%（64ms）

### 题意

传送带上有`N`个货物，每个货物的重量为`weights[i]`。每艘船每天可以运走不超过载重量的连续若干个货物。问船的载重量至少为多少，才能在`D`天内运完货物？

### 分析

看起来非常经典的一道二分答案。给定载重量和`D`，判断能否运完货物是很简单（`O(N)`）的。那么直接对载重量进行二分就可以了。

### 代码

```cpp
class Solution {
public:
    bool isOk(vector<int>& weights, int D, int C) {
        int cnt = 0;
        int loaded = 0;
        for (int w: weights) {
            // 这里加了一些特判==
            if (w > C) return false;
            if (loaded == C) {
                loaded = 0;
                cnt++;
            }
            if (loaded + w > C) {
                cnt++;
                loaded = w;
            }
            else
                loaded += w;
        }
        if (loaded != 0) cnt++;
        return cnt <= D;
    }
    
    int shipWithinDays(vector<int>& weights, int D) {
        int l = 1, r = 25000000;
        while (l < r) {
            int m = (l + r) / 2;
            if (isOk(weights, D, m)) r = m;
            else l = m + 1;
        }
        return l;
    }
};
```

## [1012. Numbers With Repeated Digits](https://leetcode.com/problems/numbers-with-repeated-digits/description/)

标记难度：Hard

提交次数：1/1

代码效率：33.79%（64ms）
