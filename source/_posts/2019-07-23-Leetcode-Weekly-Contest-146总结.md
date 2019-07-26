---
title: Leetcode Weekly Contest 146总结
urlname: leetcode-weekly-contest-146
toc: true
date: 2019-07-23 02:07:39
updated: 2019-07-27 01:42:39
tags: [Leetcode, Leetcode Contest, alg:Breadth-First Search, alg:Dynamic Programming, alg:Math]
categories: Leetcode
---

本周的第四题是道富含思维含量的好题，换句话说，就是一道让我做了一个小时仍然毫无头绪，只好去看题解，结果连题解都看不太懂的题。

<!--more-->

## [1128. Number of Equivalent Domino Pairs](https://leetcode.com/problems/number-of-equivalent-domino-pairs/description/)

标记难度：Easy

提交次数：1/1

代码效率：25.52%（60ms）

### 题意

给定一个数组的多米诺骨牌（意思是上面写着两个1-6的数字的牌，数字顺序无关），问相等的骨牌对数。

### 分析

直接搞个map统计一下每种等价的牌的数量，然后直接`C(n, 2)`就可以了。

### 代码

```cpp
class Solution {
public:
    int numEquivDominoPairs(vector<vector<int>>& dominoes) {
        map<vector<int>, int> mmap;
        for (auto d: dominoes) {
            sort(d.begin(), d.end());
            mmap[d]++;
        }
        int ans = 0;
        for (auto p: mmap) {
            ans += (p.second - 1) * p.second / 2;
        }
        return ans;
    }
};
```

## [1129. Shortest Path with Alternating Colors](https://leetcode.com/problems/shortest-path-with-alternating-colors/description/)

标记难度：Medium

提交次数：1/1

代码效率：25.52%（60ms）

### 题意

给定一个有向图（边权均为1），图中的边分成红蓝两种颜色，问从0出发到每个点的红蓝（或蓝红）交替的路径的最小长度。

### 分析

这道题的本质是拆点。把每个点都拆成一个红点和一个蓝点，认为从红点只能从红边出发（并到达蓝点），从蓝点只能从蓝边出发（并到达红点）。然后就可以进行多源点BFS了（两个源点分别是红0和蓝0）。

### 代码

```cpp
class Solution {
public:
    vector<int> shortestAlternatingPaths(int n, vector<vector<int>>& red_edges, vector<vector<int>>& blue_edges) {
        bool visited[105][2]; // 0=red, 1=blue
        int dist[105][2];
        queue<pair<int, int>> q;
        
        vector<int> G[105][2];
        for (auto& v: red_edges) {
            G[v[0]][0].push_back(v[1]);
        }
        for (auto& v: blue_edges) {
            G[v[0]][1].push_back(v[1]);
        }
        
        memset(visited, 0, sizeof(visited));
        memset(dist, 1, sizeof(dist));  // note: the longest path len can exceed n
        visited[0][0] = visited[0][1] = true;
        dist[0][0] = dist[0][1] = 0;
        q.emplace(0, 0); q.emplace(0, 1);
        
        while (!q.empty()) {
            int u = q.front().first, color = q.front().second;
            q.pop();
            
            for (int v: G[u][1 - color]) {
                if (!visited[v][1 - color]) {
                    visited[v][1 - color] = true;
                    dist[v][1 - color] = dist[u][color] + 1;
                    q.emplace(v, 1 - color);
                }
            }
        }
        
        vector<int> ans;
        for (int i = 0; i < n; i++) {
            ans.push_back(min(dist[i][0], dist[i][1]));
            if (ans.back() > 1e5)
                ans.back() = -1;
        }
        return ans;
    }
};
```

## [1130. Minimum Cost Tree From Leaf Values](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values/description/)

标记难度：Medium

提交次数：1/1

代码效率：25.52%（16ms）

### 题意

给定一个正整数数组，用它们作为叶子构造一棵每个结点的子结点数量都是0或2的二叉树，且这些数在所有叶子中从左到右依次出现。记每个非叶结点的值是它的左右子树中各自最大的叶结点的乘积，求所有合法的树中非叶结点值的和的最小值。

### 分析

这道题的题目描述简直绕上天了，但形式化出来之后会发现实际上很简单。显然，我们可以自顶向下递归构造出一棵树：每一步把数组分成左右两个部分，左边用于构造左子树，右边用于构造右子树，然后再合并起来。于是每个数组区间都可以代表一棵子树。然后就可以DP了，时间复杂度是`O(N^2)`。

### 代码

```cpp
class Solution {
public:
    int mctFromLeafValues(vector<int>& arr) {
        int n = arr.size();
        int maxn[41][41];
        int f[41][41];  // smallest node sum
        
        for (int i = 0; i < n; i++) {
            maxn[i][i] = arr[i];
            for (int j = i + 1; j < n; j++) {
                maxn[i][j] = max(maxn[i][j-1], arr[j]);
            }
        }
        
        memset(f, 0, sizeof(f));
        for (int l = 2; l <= n; l++) {
            for (int i = 0; i + l <= n; i++) {
                f[i][i + l - 1] = 1e9;
                // split: j = left count
                for (int j = 1; j < l; j++) {
                    f[i][i+l-1] = min(f[i][i+l-1], f[i][i+j-1] + f[i+j][i+l-1] + maxn[i][i+j-1] * maxn[i+j][i+l-1]);
                }
                cout << i << ' ' << i + l - 1 << ' ' << f[i][i+l-1] << endl;
            }
        }
        
        return f[0][n-1];
    }
};
```

## [1131. Maximum of Absolute Value Expression](https://leetcode.com/problems/maximum-of-absolute-value-expression/description/)

标记难度：Medium

提交次数：1/1

代码效率：69.43%（40ms）

### 题意

给定两个等长的整数数组`arr1`和`arr2`，求`|arr1[i] - arr1[j]| + |arr2[i] - arr2[j]| + |i - j|`的最大值。

### 分析

题意简明易懂，然而真不会做。最后只好跑到Discussion里找了一篇我认为最好懂的[题解](https://leetcode.com/problems/maximum-of-absolute-value-expression/discuss/340051/Python-very-intuitive-math-solution-with-explanation-O(N))，才明白应该怎么做（然而我还是不懂这怎么能想出来）。

主要思路是这样的：为简单起见，用`a`表示`arr1`，`b`表示`arr2`，设`i > j`，则

```cpp
|a[i] - a[j]| + |b[i] - b[j]| + |i - j|
= |a[i] - a[j]| + |b[i] - b[j]| + i - j
```

显然上式的取值有以下4种可能情况：

```cpp
= a[i] - a[j] + b[i] - b[j] + i - j = (a[i] + b[i] + i) - (a[j] + b[j] + j)  (a[i] >= a[j], b[i] >= b[j])
= a[i] - a[j] + b[j] - b[i] + i - j = (a[i] - b[j] + i) - (a[j] - b[j] + j)  (a[i] >= a[j], b[i] < b[j])
= a[j] - a[i] + b[i] - b[j] + i - j = (-a[i] + b[i] + i) - (-a[j] + b[j] + j) (a[i] < a[j], b[i] >= b[j])
= a[j] - a[i] + b[j] - b[i] + i - j = (-a[i] - b[i] + i) - (-a[j] - b[j] + j) (a[i] < a[j], b[i] < b[j])
```

对式子进行变化之后，可以看出，每种情况都可以被看成是两项的差，它们的形式相同，且都只与一个单独的index相关。于是就可以得出一种简单的方法：遍历数组，为每种情况的项分别求出最大值和最小值，于是整体的最大值就等于每种情况的最大值减最小值中的最大值。

于是就会产生两个问题。第一个问题是，假如找到的最大值和最小值对不满足`i > j`的要求呢？（显然，为了规避这个问题，可以把对i和j的讨论也加进去，然后就会变成八种情况；但实际上并不需要。）不妨假设对于找到的最大值和最小值，`i <= j`，则立刻可以发现`|a[i] - a[j]| + |b[i] - b[j]| + |i - j| = |a[j] - a[i]| + |b[j] - b[i]| + |j - i|`，也就是说至少会有另一对满足`i < j`的最大值和最小值。[^ij]

[^ij]: [Leetcode 1131 Discussion - What if the max/min you recorded don't align with the i < j assumption?](leetcode.com/problems/maximum-of-absolute-value-expression/discuss/340051/Python-very-intuitive-math-solution-with-explanation-O(N)/309831)

另一个问题是，如果找到的整体最大值和最小值对不满足元素大小的要求呢？以第一种情况为例，如何保证`a[i] >= a[j], b[i] >= b[j]`？可以采用反证法。不妨假设`a[i] < a[j], b[i] >= b[j]`，则此时很容易看出，`(-a[i] + b[i] + i) - (-a[j] + b[j] + j) > (a[i] + b[i] + i) - (a[j] + b[j] + j)`，这不符合取到的值对是第一种情况的要求。

### 代码

```cpp
class Solution {
public:
    int maxAbsValExpr(vector<int>& arr1, vector<int>& arr2) {
        int maxn[4], minn[4];
        for (int i = 0; i < 4; i++) {
            maxn[i] = -1;
            minn[i] = 1e9;
        }
        int n = arr1.size();
        for (int i = 0; i < n; i++) {
            int x[4];
            x[0] = arr1[i] + arr2[i] + i;
            x[1] = arr1[i] - arr2[i] + i;
            x[2] = - arr1[i] + arr2[i] + i;
            x[3] = - arr1[i] - arr2[i] + i;
            
            for (int j = 0; j < 4; j++) {
                maxn[j] = max(maxn[j], x[j]);
                minn[j] = min(minn[j], x[j]);
            }
        }
        
        int ans = -1;
        for (int j = 0; j < 4; j++)
            ans = max(ans, maxn[j] - minn[j]);
        return ans;
    }
};
```
