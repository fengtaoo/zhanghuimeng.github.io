---
title: Leetcode Weekly Contest 136总结
urlname: leetcode-weekly-contest-136
toc: true
mathjax: true
date: 2019-05-12 01:14:56
updated: 2019-05-13 21:12:00
tags: [Leetcode, Leetcode Contest, alg:Binary Search, alg:Depth-first Search, alg:Rabin-Karp, alg:Suffix Array]
categories: Leetcode
---

我觉得这场比赛最难的是第一道题……

<!--more-->

## [1041. Robot Bounded In Circle](https://leetcode.com/problems/robot-bounded-in-circle/description/)

标记难度：Easy

提交次数：1/1

代码效率：61.81%（4ms）

### 题意

给定一个方格图上的一个移动序列（包括移动和旋转两种操作），问该移动序列是否会产生循环（或者说回到原点之类的）。

### 分析

……比赛的时候看到这道题我就蒙了。

原题中问的是“never leaves the circle”，也就是机器人的移动是否是有界的；不过很显然这个可以翻译成是否必然会重复经过某个方格且行进方向重复，否则移动就不会是有界的了。从这个思路来想，最暴力的方法就是多重复走几轮，看会不会发生重复；如果发生了则必然会产生循环，如果不发生呢……？

事实上，上面这个思路是对的，而且最多走4轮就够了。具体参见[\[Java/C++/Python\] Let Chopper Help Explain](https://leetcode.com/problems/robot-bounded-in-circle/discuss/290856/JavaC++Python-Let-Chopper-Help-Explain)里的说明。如果非要严格证明的话，可以考虑每走一次时位移和相对方向的变化，不过这实在是比较麻烦。

### 代码

```cpp
class Solution {
public:
    bool isRobotBounded(string instructions) {
        int N = instructions.size();
        int mx[4] = {0, 1, 0, -1};
        int my[4] = {1, 0, -1, 0};
        int x = 0, y = 0, d = 0;
        for (int i = 0; i < N * 4; i++) {
            if (instructions[i % N] == 'L')
                d = (d + 3) % 4;
            else if (instructions[i % N] == 'R')
                d = (d + 1) % 4;
            else
                x = x + mx[d], y = y + my[d];
        }
        return x == 0 && y == 0;
    }
};
```

## [1042. Flower Planting With No Adjacent](https://leetcode.com/problems/flower-planting-with-no-adjacent/description/)

标记难度：Easy

提交次数：1/1

代码效率：80.35%（168ms）

### 题意

给定一个全部结点度数都<4的无向图，要求对该图的结点进行4-染色，使得没有两个相邻结点的颜色是相同的。

### 分析

我最开始愣了一下，然后发现是4-染色，所以直接DFS染色就行了，一定可以染出一种可行解的……

P.S. 我觉得这个第二题可比第一题要简单。

### 代码

```cpp
class Solution {
private:
    vector<int> G[10001];
    vector<int> color;
    
    void dfs(int x) {
        bool found[5];
        memset(found, 0, sizeof(found));
        for (int y: G[x])
            if (color[y - 1] != 0)
                found[color[y - 1]] = true;
        for (int i = 1; i <= 4; i++)
            if (!found[i]) {
                color[x - 1] = i;
                break;
            }
        for (int y: G[x])
            if (color[y - 1] == 0)
                dfs(y);
    }
    
public:
    vector<int> gardenNoAdj(int N, vector<vector<int>>& paths) {
        color = vector<int>(N, 0);
        for (int i = 0; i < paths.size(); i++) {
            int x = paths[i][0], y = paths[i][1];
            G[x].push_back(y);
            G[y].push_back(x);
        }
        // 注意这个图可能不连通……
        for (int i = 1; i <= N; i++)
            if (color[i - 1] == 0)
                dfs(i);
        return color;
    }
};
```

## [1043. Partition Array for Maximum Sum](https://leetcode.com/problems/partition-array-for-maximum-sum/description/)

标记难度：Medium

提交次数：1/1

代码效率：36.60%（28ms）

### 题意

将数组`A`分成若干个连续子序列，每个子序列的长度不超过`K`，求（每个子序列长度×子序列中最大值）的和的最大值。

### 分析

这就是很传统的DP题了。令`f[i] = maxsum(0, 1, ..., i)`，则`f[i] = max(f[i - j] + maxn(f[i-j+1 ... i]) * j) (1 <= j <= K)`。预处理子序列的最大值可以将时间复杂度降低到`O(NK)`。

### 代码

```cpp
class Solution {
private:
    int f[501];
    int maxn[501][501];  // [i, i+j]内的最大值
    int N;
public:
    int maxSumAfterPartitioning(vector<int>& A, int K) {
        N = A.size();
        for (int i = 0; i < N; i++) {
            maxn[i][0] = A[i];
            for (int j = 1; j < K && i + j < N; j++)
                maxn[i][j] = max(maxn[i][j-1], A[i+j]);
        }
        for (int i = 0; i < N; i++) {
            f[i] = -1;
            for (int j = 1; j <= K && j <= i + 1; j++) {
                int x = maxn[i - j + 1][j - 1] * j;
                if (i - j >= 0) x += f[i - j];
                f[i] = max(f[i], x);
            }
        }
        return f[N - 1];
    }
};
```

## [1043. Partition Array for Maximum Sum](https://leetcode.com/problems/partition-array-for-maximum-sum/description/)

标记难度：Medium

提交次数：1/1

代码效率：36.60%（28ms）

### 题意

给定字符串`S`，求出`S`中最长的重复出现的连续子串的长度。（子串可以重叠）

### 分析

这道题和[P2743 \[USACO5.1\]乐曲主题Musical Themes](https://www.luogu.org/problemnew/show/P2743)有点像，除了重复出现的定义之外。事实上有三种经典的做法：

* DP：这个idea是很显然的，但复杂度是`O(N^2)`，不适合这道题
* 二分+RK算法：容易想到，复杂度最好时为`O(N*log(N))`
* 后缀数组：复杂度为`O(N*log(N))`（但我不会写）

所以就先二分答案+RK算法好了。

显然，如果有长度为`n`的连续子串，那么也必然有长度为`n-1`的连续子串，因此二分答案是可行的。接下来的重点就是判断，是否有长度为`n`的重复子串。这个应用场景就很适合[Rabin_Karp算法](https://en.wikipedia.org/wiki/Rabin%E2%80%93Karp_algorithm)了：

{% raw %}
$$
\begin{aligned}
h_i &= (d^{m-1} * s_{i} + d^{m-2} * s_{i+1} + \cdots d * s_{i+m-2} + s_{i+m-1}) \bmod p \\
h_{i+1} &= (d^{m} * s_{i+1} + d^{m-1} * s_{i+2} + \cdots d * s_{i+m-1} + s_{i+m}) \bmod p \\
&= ((h_i - d^{m-1} * s{i}) * d + s_{i+m}) \bmod p
\end{aligned}
$$
{% endraw %}

对长度`n`进行二分，然后计算每个长度为$n$的子串的哈希值，然后检查其中是否有碰撞，并根据结果调整二分的范围。

### 代码

```cpp
class Solution {
private:
    typedef long long LL;
    LL h[100001];
    string S;
    int N;
    string ans;
    
    LL P = 1e9 + 7, d = 256;
    
    // 计算长度为len时的哈希值
    void calc(int len) {
        h[0] = 0;
        LL pow = 1;
        for (int i = 0; i < len; i++) {
            h[0] = (h[0] * d + S[i]) % P;
            if (i != 0) pow = (pow * d) % P;
        }
        for (int i = 1; i + len <= N; i++) {
            h[i] = ((h[i-1] - S[i-1] * pow) % P + P) % P;
            h[i] = (h[i] * d + S[i + len - 1]) % P;
        }
    }
    
    bool checkIsSame(int x, int y, int len) {
        for (int i = 0; i < len; i++)
            if (S[x + i] != S[y + i]) return false;
        return true;
    }
    
    bool isOk(int len) {
        calc(len);
        // 判断是否发生碰撞
        unordered_map<int, vector<int>> hashMap;
        for (int i = 0; i + len <= N; i++) {
            if (hashMap.find(h[i]) != hashMap.end()) {
                for (int x: hashMap[h[i]])
                    if (checkIsSame(i, x, len)) {
                        if (len > ans.size())
                            ans = S.substr(i, len);
                        return true;
                    }
            }
            hashMap[h[i]].push_back(i);
        }
        return false;
    }
    
public:
    string longestDupSubstring(string S) {
        this->S = S;
        N = S.length();
        int l = 0, r = S.size();
        while (l < r - 1) {
            int m = (l + r) / 2;
            if (isOk(m)) l = m;
            else r = m - 1;
            cout << ans << endl;
        }
        if (isOk(l + 1)) l++;
        return ans;
    }
};
```
