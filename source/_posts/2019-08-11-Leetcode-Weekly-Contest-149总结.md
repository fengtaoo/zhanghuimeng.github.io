---
title: Leetcode Weekly Contest 149总结
urlname: leetcode-weekly-contest-149
toc: true
date: 2019-08-11 16:35:45
updated: 2019-08-11 16:35:45
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming, alg:Index Search Array]
categories: Leetcode
---

这周的最后一题可以说是非常有趣，我从中学到了很多新的idea。

<!--more-->

## [1154. Day of the Year](https://leetcode.com/problems/day-of-the-year/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms

### 题意

给定一个形如`YYYY-MM-DD`的日期字符串，问这个日期是这一年的第几天。

### 分析

直接从之前的月份开始遍历，把每月的天数加起来即可。

所以这其实是一道考闰年的题。

### 代码

```cpp
class Solution {
public:
    int ordinalOfDate(string date) {
        int year = stoi(date.substr(0, 4));
        int month = stoi(date.substr(5, 2));
        int day = stoi(date.substr(8, 2));

        for (int i = 1; i < month; i++) {
            if (i == 1 || i == 3 || i == 5 || i == 7 || i == 8 || i == 10 || i == 12)
                day += 31;
            else if (i == 2) {
                if (year % 4 == 0 && (year % 100 != 0) || year % 400 == 0)
                    day += 29;
                else
                    day += 28;
            }
            else
                day += 30;
        }
        return day;
    }
};
```

## [1155. Number of Dice Rolls With Target Sum](https://leetcode.com/problems/number-of-dice-rolls-with-target-sum/description/)

标记难度：Medium

提交次数：1/5

代码效率：36ms

### 题意

给定`d`个骰子，每个骰子有`f`面，分别标着`1...f`。问用这些骰子的顶面组成`target`的方法数量？

### 分析

可以进行DP：令`g[i][j]`表示用前`i`个骰子组成`j`的方法总数，则可以立即写出转移方程：

```cpp
g[i][j] = sum(g[i-1][j-k]) (1 <= k <= f and k <= j)
```

总的来说，这是一道比较简单的DP题。

### 代码

```cpp
class Solution {
public:
    int numRollsToTarget(int d, int f, int target) {
        typedef long long LL;
        LL g[31][1005];  // 用前i个骰子组成j的方法总数
        LL P = 1e9 + 7;
        memset(g, 0, sizeof(g));
        g[0][0] = 1;
        for (int i = 1; i <= d; i++) {
            for (int j = 1; j <= target; j++) {
                for (int k = 1; k <= f && k <= j; k++) {
                    g[i][j] += g[i - 1][j - k];
                    g[i][j] %= P;
                }
            }
        }
        return (int) g[d][target];
    }
};
```

## [1156. Swap For Longest Repeated Character Substring](https://leetcode.com/problems/swap-for-longest-repeated-character-substring/description/)

标记难度：Medium

提交次数：1/3

代码效率：36ms

### 题意

给定一个字符串，问字符串中最长的连续字符序列和进行任意一次字符交换后的最长连续字符序列的最大长度？

### 分析

直接求字符串中的最长连续字符序列是平凡的，用最大连续子序列的思路即可。问题是“交换”到底意味着什么。以下面这个字符串片段为例：

```txt
....AAAAXAA...
```

显然有一种方法使得交换后连续序列的长度变长，就是把最靠左的`A`或者最靠右的`A`和中间的`X`交换：

```txt
....XAAAAAA...
....AAAAAAX...
```

如果字符串其他位置还存在其他的`A`，可以把它和`X`交换，得到更长的连续序列：

```txt
....AAAAXAA...A...
....AAAAAAA...X...
```

当然，这么做的前提是存在，因此要通过计数的方法判断一下当前考虑的两个连续序列之外还有没有别的`A`了。

最后一种情况是上一种情况的变体：

```txt
....XAAAAX...A...
....XAAAAA...X...
```

### 代码

直接算最长连续子序列的方法不如直接先把相同的字符合并，不过C++写起来未必像Python[^groupby]那样优雅。

[^groupby]: [lee215's Solution for Leetcode 1156 - Python Groupby](https://leetcode.com/problems/swap-for-longest-repeated-character-substring/discuss/355852/Python-Groupby)

```cpp
class Solution {
public:
    int maxRepOpt1(string text) {
        int cnt[26];
        memset(cnt, 0, sizeof(cnt));
        int leftMax[20005], rightMax[20005];
        
        // 计算向左和向右延伸的最长连续子序列
        int n = text.size();
        for (int i = 0; i < n; i++)
            cnt[text[i] - 'a']++;
        for (int i = 0; i < n; i++) {
            if (i == 0 || text[i] != text[i - 1])
                leftMax[i] = 1;
            else
                leftMax[i] = leftMax[i-1] +1;
        }
        for (int i = n - 1; i >= 0; i--) {
            if (i == n - 1 || text[i] != text[i+1])
                rightMax[i] = 1;
            else
                rightMax[i] = rightMax[i+1] + 1;
        }
        
        int ans = 1;
        for (int i = 0; i < n; i++) {
            // 没有交换的情况
            ans = max(leftMax[i], ans);
            ans = max(rightMax[i], ans);
            if (i == 0 || i == n - 1) continue;
            if (text[i-1] == text[i+1] && text[i-1] != text[i]) {
                // 第二种情况
                if (cnt[text[i-1]-'a'] > leftMax[i-1] + rightMax[i+1])
                    ans = max(ans, 1 + leftMax[i-1] + rightMax[i+1]);
                // 第一种情况
                else
                    ans = max(ans, leftMax[i-1] + rightMax[i+1]);
            }
            // 第三种情况
            else {
                if (cnt[text[i-1] - 'a'] > leftMax[i-1]) ans = max(leftMax[i] + 1, ans);
                if (cnt[text[i+1] - 'a'] > rightMax[i+1]) ans = max(rightMax[i] + 1, ans);
            }
        }
        
        return ans;
    }
};
```

## [1157. Online Majority Element In Subarray](https://leetcode.com/problems/online-majority-element-in-subarray/description/)

标记难度：Hard

提交次数：1/1

代码效率：

* 随机法：71.60%（212ms）

### 题意

### 分析

一位[zerotrac2](https://leetcode.com/zerotrac2/)对这道题给出了一些极好的[分析](https://leetcode.com/problems/online-majority-element-in-subarray/discuss/356227/C++-Codes-of-different-approaches-(Random-Pick-Trade-off-Segment-Tree-Bucket))，比我所知的要强得多，因此我决定把他的分析翻译一下，从中学习一些内容。

可以说这道题的核心就是这个“majorty”，因为有这一要求，所以很多时间复杂度分析都能成立。

#### Index Search Array

其实我也不知道这个结构的学名到底是不是Index Search Array，但总之，对于一个数组，把其中每个元素及其出现的位置都按顺序存储到一个`map`中，你就得到了这个结构；然后通过二分查找，就可以很方便地查询任意区间内某个元素的出现次数了。

#### Boyer–Moore majority vote algorithm

这个算法是求区间内的majority的（注意，并不是众数）；如果majority不存在，它也会输出一个数，而不会告诉你没有majority。算法的思路超简单：假设区间内的majority存在，记为`x`，则我们把所有不是`x`的数都找一个`x`和它一起消去，最后肯定还剩一些`x`。

算法伪代码如下[^wiki]：

[^wiki]: [Wikipedia - Boyer–Moore majority vote algorithm](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm)

```py
m = NaN  # 记录元素
i = 0    # 记录出现次数
for x in A:
    if i == 0:
        m = x
        i = 1
    elif m == x:
        i += 1
    else:
        i -= 1
return m
```

#### 随机法

给定一个区间，显然可以尝试从中随机选择数字，然后判断该数字是否是该区间中的majority（需要用到Index Search Array）。如果经过多次随机都找不到，则说明不存在majority。通过多次随机，可以将错误概率降低到非常小，可以算是一种成功的算法。

#### Trade-off

简单来说，就是在不同查询区间长度对应的时间复杂度之间进行Trade-off。之所以能这么做，是基于下面这个事实：令`t`表示出现频率的阈值，则在整体数组中出现次数大于`t`的数不超过`N / t`个。如果当前查询的阈值`threshold >= t`，则我们只需考虑前面所说的那些数字。

#### 线段树

可以推理出，如果一个区间有majority，那么把它均分之后，一定至少有一个区间的majority还是它。因此可以采用线段树算法，建树时合并两个区间的majority时直接用Index Search Array；在查询时，也可以对找到的所有区间的majority都查询总出现次数。

#### 桶

其实这也是一种trade-off的方法，而且思路有一点类似线段树。将整个区间划分成t个桶，每个桶的大小为`n/t`；预先计算出每个桶的majority，这样在查询的时候就只需要检查最多`n/t`个数是否为majority了（其中有最多两个边上的数是现算的）。

### 代码

只写得动随机法了，其他的不写了……

#### 随机法

```cpp
class MajorityChecker {
    unordered_map<int, vector<int>> indexArr;
    vector<int> arr;
    int n;
    
    // Index Search Array初始化
    void buildIndexArr() {
        for (int i = 0; i < n; i++) {
            indexArr[arr[i]].push_back(i);
        }
    }
    
    // Index Search Array在[l, r]区间内查询
    int countInterval(int l, int r, int ele) {
        auto iter = indexArr.find(ele);
        if (iter == indexArr.end()) return 0;
        auto up_iter = upper_bound(iter->second.begin(), iter->second.end(), r);
        auto lo_iter = lower_bound(iter->second.begin(), iter->second.end(), l);
        return up_iter - lo_iter;
    }
    
public:
    MajorityChecker(vector<int>& arr) {
        this->arr = arr;
        n = arr.size();
        buildIndexArr();
        srand(time(0));
    }
    
    int query(int left, int right, int threshold) {
        // 每次在区间内随机选，选20次，错误概率较小
        for (int i = 0; i < 20; i++) {
            int idx = rand() % (right - left + 1) + left;
            int cnt = countInterval(left, right, arr[idx]);
            if (cnt >= threshold)
                return arr[idx];
        }
        return -1;
    }
};
```
