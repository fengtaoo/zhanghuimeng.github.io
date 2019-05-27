---
title: Leetcode Weekly Contest 138总结
urlname: leetcode-weekly-contest-138
toc: true
date: 2019-05-26 20:39:18
updated: 2019-05-27 16:53:00
tags: [Leetcode, Leetcode Contest, alg:Greedy, alg:Dynamic Programming, alg:Monotonic Stack]
categories: Leetcode
---

我觉得第3题最难。但是，好像比赛的时候第3题出锅了呢……

<!--more-->

## [1051. Height Checker](https://leetcode.com/problems/height-checker/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms

### 题意

给定一个数组，问其中没有正确归到排序后的位置的数有多少个。

### 分析

直接复制一个数组，排个序，和原来的数组比较一下就可以了。

### 代码

```cpp
class Solution {
public:
    int heightChecker(vector<int>& heights) {
        vector<int> sorted(heights);
        sort(sorted.begin(), sorted.end());
        int ans = 0;
        for (int i = 0; i < heights.size(); i++)
            if (heights[i] != sorted[i])
                ans++;
        return ans;
    }
};
```

## [1052. Grumpy Bookstore Owner](https://leetcode.com/problems/grumpy-bookstore-owner/)

标记难度：Easy

提交次数：2/3

代码效率：36ms

### 题意

给定一个整数数组，其中的一些位置被标记了。可以将连续`X`个位置的标记消去（只能消去一次）。问消去后未被标记的整数之和最大是多少？

### 分析

最暴力的做法是枚举每个开始消去的位置，复杂度是`O(N*X)`，但以本题`X, N <= 20000`的数据规模显然是不行的。解决方案也很简单，计算原来所有没有被标记的整数的和，然后只算出在每个位置消去会增加的和，把两个加在一起取最大值就行了。而“每个位置消去会增加的和”这个东西可以从前往后递推：

```cpp
more[i+1] = more[i] - grumpy[i-1] * customers[i-1] + grumpy[i+X-1] * customers[i+X-1]
```

复杂度是`O(N+X)`。

### 代码

```cpp
class Solution {
public:
    int maxSatisfied(vector<int>& customers, vector<int>& grumpy, int X) {
        int more[20005];
        int n = customers.size();
        
        int sum = 0;
        for (int i = 0; i < n; i++)
            if (!grumpy[i])
                sum += customers[i];
        
        int ans = 0;
        more[0] = 0;
        for (int i = 0; i < X && i < n; i++)
            if (grumpy[i])
                more[0] += customers[i];
        ans = max(ans, sum + more[0]);
        for (int i = 1; i < n; i++) {
            more[i] = more[i-1] - grumpy[i-1] * customers[i-1];
            if (i + X - 1 < n)
                more[i] += grumpy[i+X-1] * customers[i+X-1];
            ans = max(ans, sum + more[i]);
        }
        
        return ans;
    }
};
```

## [1053. Previous Permutation With One Swap](https://leetcode.com/problems/previous-permutation-with-one-swap/description/)

标记难度：Medium

提交次数：1/1

代码效率：112ms

### 题意

给定一个排列（可能有重复元素），求出只通过一次元素交换能得到的最大的比当前排列小的元素。

### 分析

比赛的时候我认为这道题很难，所以根本就没看题。后来发现并不难——显然，如果交换一次元素（不妨记为`(A[i], A[j])`）之后得到的元素比当前排列小，那么必然有`A[i] > A[j]`。而且对于`A[j]`来说，`i`越大，得到的新排列越大（因为`A[j]`前面的其他元素都不变，那么变小的元素越靠后，得到的新排列就相应越大）。所以这道题就转换成了一个单调栈问题，对每个元素求出左侧比它大的最靠右的元素（不妨记为`maxIdx[j]`）。

求出来之后还需要一些讨论。对于不同的`j`，显然，`maxIdx[j]`越大，得到的新排列越大；而当`maxIdx[j1] == maxIdx[j2]`时，考虑以下例子：

```cpp
[1, 2, 5, 3, 3, 4]
j1=3 -> [1, 2, 3, 5, 3, 4]
j2=4 -> [1, 2, 3, 3, 5, 4]
```

显然选择较小的`j`得到的新排列更大。

### 代码

```cpp
class Solution {
public:
    vector<int> prevPermOpt1(vector<int>& A) {
        stack<pair<int, int>> s;
        int maxIdx[10005];
        int n = A.size();
        for (int i = n - 1; i >= -1; i--) {
            int x = i >= 0 ? A[i] : 1e9;
            if (s.empty() || x <= s.top().first) {
                s.emplace(x, i);
            }
            else {
                while (!s.empty() && x > s.top().first) {
                    maxIdx[s.top().second] = i;
                    s.pop();
                }
                s.emplace(x, i);
            }
        }
        int idx = -1, minVal;
        for (int i = n - 1; i >= 0; i--) {
            if (maxIdx[i] == -1) continue;
            if (idx == -1) {
                idx = i;
                minVal = A[i];
                continue;
            }
            if (maxIdx[i] < maxIdx[idx]) continue;
            else if (maxIdx[i] > maxIdx[idx] || maxIdx[i] == maxIdx[idx] && A[i] >= minVal) {
                idx = i;
                minVal = A[i];
            }
        }
        // 注意无解的情况
        if (idx != -1)
            swap(A[maxIdx[idx]], A[idx]);
        return A;
    }
};
```

## [1054. Distant Barcodes](https://leetcode.com/problems/distant-barcodes/description/)

标记难度：Medium

提交次数：1/1

代码效率：304ms

### 题意

给定`N`个整数，将这`N`个整数排成一排，使得没有两个相邻的数是相等的。

### 分析

我之前在CodeForces上做过这道题。下面是一个比较简单的贪心思路：统计每种整数的数量，放到一个以数量为关键字的堆里，然后每次取出堆顶元素放到结果数组中，将其数量减1（如果取出来的恰好是上一个取过的数就再取一个，然后把多取的再放回去）。复杂度是`O(N*log(N))`。

另一个思路是直接按照出现次数从多到少，把整数先放到结果数组的奇数位上，再放到偶数位上。不过这种思路的复杂度似乎没有降低，我也懒得去写了。[^lee]

[^lee]: [lee215's Solution for Leetcode 1054 - \[Python\] Set Odd Position and  Even Position](https://leetcode.com/problems/distant-barcodes/discuss/299225/Python-Set-Odd-Position-and-Even-Position)

### 代码

```cpp
class Solution {
public:
    vector<int> rearrangeBarcodes(vector<int>& packages) {
        unordered_map<int, int> hMap;
        for (int p: packages)
            hMap[p]++;
        priority_queue<pair<int, int>> pq;
        for (auto& p: hMap)
            pq.emplace(p.second, p.first);
        vector<int> ans;
        while (!pq.empty()) {
            int cnt1 = pq.top().first, num1 = pq.top().second;
            pq.pop();
            if (!ans.empty() && ans.back() == num1) {
                int cnt2 = pq.top().first, num2 = pq.top().second;
                pq.pop();
                ans.push_back(num2);
                cnt2--;
                if (cnt2 > 0)
                    pq.emplace(cnt2, num2);
            }
            else {
                ans.push_back(num1);
                cnt1--;
            }
            if (cnt1 == 0) continue;
            pq.emplace(cnt1, num1);
        }
        return ans;
    }
};
```
