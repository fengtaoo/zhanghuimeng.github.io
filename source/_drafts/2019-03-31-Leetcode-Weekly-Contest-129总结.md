---
title: Leetcode Weekly Contest 129总结
urlname: leetcode-weekly-contest-129
toc: true
date: 2019-03-31 21:38:32
updated: 2019-03-31 21:38:32
tags: [Leetcode, Leetcode Contest]
categories: Leetcode
---

这周的比赛稍微有点难度。然而我睡过了头，一道也没做==

<!--more-->

## [1013. Partition Array Into Three Parts With Equal Sum](https://leetcode.com/problems/complement-of-base-10-integer/description/)

标记难度：Easy

提交次数：1/1

代码效率：

* 愚蠢的方法：0.00%（168ms）
* 智慧的方法：13.85%（84ms）

### 题意

给定整数数组`A`，问能否将`A`分成三段，这三段的和都相等。

## 分析

我的做法比较愚蠢。首先判断`sum(A)`是否模3余0，如果不是的话，显然不存在这样的分割。然后我找出所有和为`sum(A) / 3`和`sum(A) * 2 / 3`的前缀数组，只要存在一个`sum(A) * 2 / 3`的前缀比`sum(A) / 3`的前缀靠后，就存在这样的分割。

这个做法有点儿蠢。得知了`sum(A)`后，可以直接开始找和等于`sum(A) / 3`的前缀。如果能找到，把它删掉，再找下一个这样的前缀。最后判断找到的总数是否为3（注意为0的情形）。[^vortubac]

[^vortubac]: [vortubac's Solution for Leetcode 1013 - C++ 6 lines O(n), target sum](https://leetcode.com/problems/partition-array-into-three-parts-with-equal-sum/discuss/260885/C++-6-lines-O(n)-target-sum)

## 代码

### 愚蠢的方法

```cpp
class Solution {
public:
    bool canThreePartsEqualSum(vector<int>& A) {
        int n = A.size();
        int sum[50005];
        map<int, vector<int>> sumMap;
        for (int i = 0; i < n; i++) {
            sum[i] = i == 0 ? A[0] : sum[i-1] + A[i];
            sumMap[sum[i]].push_back(i);
        }
        int all = sum[n - 1];
        if (all % 3 != 0) return false;
        int part = all / 3;
        if (sumMap[part].size() == 0 || sumMap[part * 2].size() == 0) return false;
        return !(sumMap[part * 2].back() <= sumMap[part][0]);
    }
};
```

### 智慧的方法

```cpp
class Solution {
public:
    bool canThreePartsEqualSum(vector<int>& A) {
        int sum = 0;
        for (int x: A) sum += x;
        if (sum % 3 != 0) return false;
        int target = sum / 3;
        int cnt = 0;
        sum = 0;
        for (int x: A) {
            sum += x;
            if (sum == target) {
                cnt++;
                sum = 0;
            }
        }
        return target == 0 ? cnt >= 3 : cnt == 3;
    }
};
```

## [1015. Smallest Integer Divisible by K](https://leetcode.com/problems/smallest-integer-divisible-by-k/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* 愚蠢的方法：0.00%（168ms）