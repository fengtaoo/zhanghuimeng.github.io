---
title: Leetcode Weekly Contest 133总结
urlname: leetcode-weekly-contest-133
toc: true
date: 2019-04-22 00:19:52
updated: 2019-04-28 19:54:52
tags: [Leetcode, alg:Greedy, alg:Trie, alg:Aho–Corasick Algorithm]
categories: Leetcode
---

我对于以后面试的时候不需要默写AC自动机如同不需要默写KMP一样自信，不需要默写KMP如同不需要默写快排一样自信，换句话说，就是我一点也不自信。也许以后还得当场讲清红黑树插入的几种情况呢。

<!--more-->

## [1030. Matrix Cells in Distance Order](https://leetcode.com/problems/matrix-cells-in-distance-order/description/)

标记难度：Easy

提交次数：1/1

代码效率：132ms

### 题意

给定一个`R`行`C`列方格图，把其中的所有方格按照到坐标`(r0, c0)`的曼哈顿距离排序。

### 分析

当然可以把所有方格枚举出来然后排个序；不过完全没有这样的必要，可以直接在`(r0, c0)`周围绕圈。

### 代码

不过还是直接排序来得快。

```cpp
class Solution {
public:
    vector<vector<int>> allCellsDistOrder(int R, int C, int r0, int c0) {
        vector<vector<int>> ans;
        for (int i = 0; i < R; i++)
            for (int j = 0; j < C; j++) {
                    int d = abs(i - r0) + abs(j - c0);
                    ans.push_back({d, i, j});
                }
        sort(ans.begin(), ans.end());
        for (int i = 0; i < ans.size(); i++)
            ans[i].erase(ans[i].begin());
        return ans;
    }
};
```

## [1029. Two City Scheduling](https://leetcode.com/problems/two-city-scheduling/description/)

标记难度：Easy

提交次数：1/1

代码效率：

* DP：8ms
* 贪心：4ms

### 题意

共有`2N`个人要坐飞机去A城或B城，给定每个人去每个城市的代价，求恰有`N`个人去A时的最小代价。

### 分析

题面看起来像一道典型的贪心问题，虽然我比赛的时候并没有意识到，于是首先写了一个DP：`f[n][m]`表示在前`n`个人中共有`m`个人去`A`城时的最小代价。转移方程是`f[n][m] = min(f[n-1][m] + cost[n][1], f[n-1][m-1] + cost[n][0])`。注意下标。

贪心的思路也很简单：不妨假设随便分配了`N`个人去A城，然后对这个解加以优化：显然，如果去A城的人中存在`i`，去B城的人中存在`j`，使得`cost[i][0] + cost[j][1] > cost[i][1] + cost[j][0]`，即`cost[i][0] - cost[i][1] > cost[j][0] - cost[j][1]`，则应该把这两个人交换一下。结论就是，把所有人按`cost[i][0] - cost[i][1]`升序排序，然后取前一半去A城就行了。

### 代码

#### DP

```cpp
class Solution {
public:
    int twoCitySchedCost(vector<vector<int>>& costs) {
        int f[105][105]; // f[n][m]: m people goes to A in the first n people
        int n = costs.size();
        memset(f, 0, sizeof(f));
        for (int i = 1; i <= n; i++) {
            f[i][0] = f[i-1][0] + costs[i-1][1];
            for (int j = 1; j <= i; j++) {
                f[i][j] = 1e9;
                // 注意这个条件==
                if (i - 1 >= j) f[i][j] = min(f[i][j], f[i-1][j] + costs[i-1][1]);
                f[i][j] = min(f[i][j], f[i-1][j-1] + costs[i-1][0]);
            }
        }
        return f[n][n / 2];
    }
};
```

#### 贪心

```cpp
class Solution {
public:
    int twoCitySchedCost(vector<vector<int>>& costs) {
        // sort in asending order of A - B
        vector<pair<int, int>> indices;
        int n = costs.size();
        for (int i = 0; i < n; i++)
            indices.emplace_back(costs[i][0] - costs[i][1], i);
        sort(indices.begin(), indices.end());
        int ans = 0;
        for (int i = 0; i < n / 2; i++)
            ans += costs[indices[i].second][0];
        for (int i = n / 2; i < n; i++)
            ans += costs[indices[i].second][1];
        return ans;
    }
};
```

## [1031. Maximum Sum of Two Non-Overlapping Subarrays](https://leetcode.com/problems/maximum-sum-of-two-non-overlapping-subarrays/description/)

标记难度：Medium

提交次数：1/1

代码效率：57.11%（12ms）

### 题意

给定数组`A`，返回`A`中两个长度分别为`L`和`M`的不相交子数组的和的最大值。

### 分析

我觉得这道题还是很简单的。

对`A`中的每个index，求出它左边的长度为`L`和`M`的子数组的和的最大值，以及它右边的长度为`L`和`M`的子数组的和的最大值，然后即可求出左边长度为`L`的子数组+右边长度为`M`的子数组的和的最大值，以及左边长度为`M`的子数组+右边长度为`L`的子数组的和的最大值。

### 代码

```cpp
class Solution {
private:
    int prefix[1005], maxLbefore[1005], maxLafter[1005], maxMbefore[1005], maxMafter[1005];
    // 求子数组和
    int get(int l, int r) {
        int ans = prefix[r];
        if (l > 0) ans -= prefix[l - 1];
        return ans;
    }
    
public:
    int maxSumTwoNoOverlap(vector<int>& A, int L, int M) {
        int n = A.size();
        for (int i = 0; i < n; i++) {
            prefix[i] = i == 0 ? 0 : prefix[i-1];
            prefix[i] += A[i];
        }
        // <=
        for (int i = 0; i < n; i++) {
            maxLbefore[i] = i == 0 ? -1 : maxLbefore[i-1];
            if (i >= L - 1)
                maxLbefore[i] = max(maxLbefore[i], get(i - L + 1, i));
            maxMbefore[i] = i == 0 ? -1 : maxMbefore[i-1];
            if (i >= M - 1)
                maxMbefore[i] = max(maxMbefore[i], get(i - M + 1, i));
        }
        // >=
        for (int i = n - 1; i >= 0; i--) {
            maxLafter[i] = i == n - 1 ? -1 : maxLafter[i+1];
            if (i + L <= n)
                maxLafter[i] = max(maxLafter[i], get(i, i + L - 1));
            maxMafter[i] = i == n - 1 ? -1 : maxMafter[i+1];
            if (i + M <= n)
                maxMafter[i] = max(maxMafter[i], get(i, i + M - 1));
        }
        int ans = 0;
        for (int i = 0; i < n - 1; i++) {
            ans = max(ans, maxLbefore[i] + maxMafter[i + 1]);
            ans = max(ans, maxMbefore[i] + maxLafter[i + 1]);
        }
        return ans;
    }
};
```

## [1032. Stream of Characters](https://leetcode.com/problems/stream-of-characters/description/)

标记难度：Hard

提交次数：1/1

代码效率：

* Trie：27.69%（536ms）
* AC自动机：没写

### 题意

事先给定一个词表，然后给你一个字符流，在中间任意时刻查询，要求返回该字符流当前末尾能否与词表中某一个词匹配。

词表大小为1-2000，词的长度为1-2000，查询次数在40000次内。

### 分析

在知道可以用Trie之后，这道题其实很简单：首先造个Trie，把所有词都倒过来之后插进去；然后对字符流也倒着处理。这样做的主要原因就是，题目要求返回的是字符流的后缀能否与词匹配，这和Trie使用前缀的本质正好是相反的，那就全都反着做好了。这样每次查询的复杂度就是线性的了。

至于[AC自动机的那些做法](https://leetcode.com/problems/stream-of-characters/discuss/278842/C++-AC-Automaton-Solution-%28Amortized-time-complexity-O%281%29%29)，实在是太过分了（虽然在这道题下是更适当的应用场景），如果我以后得做默写AC自动机的面试题，那我恐怕很快就要毕业即失业了。当然，了解一下是很好的。

### 代码

```cpp
class StreamChecker {
    struct TrieNode {
        bool isEnd;
        TrieNode* child[26];
        TrieNode() {
            isEnd = false;
            memset(child, 0, sizeof(child));
        }
    };
    
    struct Trie {
        TrieNode* root;
        Trie() {
            root = new TrieNode();
        }
        
        void insert(const string& s) {
            TrieNode* p = root;
            for (char c: s) {
                if (p->child[c - 'a'] == NULL)
                    p->child[c - 'a'] = new TrieNode();
                p = p->child[c - 'a'];
            }
            p->isEnd = true;
        }
        
        bool find(const string& s) {
            TrieNode* p = root;
            for (char c: s) {
                if (p->child[c - 'a'] == NULL)
                    return false;
                p = p->child[c - 'a'];
                if (p->isEnd) return true;
            }
            return false;
        }
    };
    
    Trie trie;
    string q;
    
public:
    StreamChecker(vector<string>& words) {
        for (string word: words) {
            reverse(word.begin(), word.end());
            trie.insert(word);
        }
    }
    
    bool query(char letter) {
        q = letter + q;
        if (q.length() > 2000) q = q.substr(0, 2000);
        return trie.find(q);
    }
};
```
