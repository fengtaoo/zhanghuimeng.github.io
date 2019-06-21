---
title: Leetcode Weekly Contest 141总结
urlname: leetcode-weekly-contest-141
toc: true
date: 2019-06-17 09:44:34
updated: 2019-06-21 19:57:00
tags: [Leetcode, Leetcode Contest, alg:Sort, alg:Breadth-first Search, alg:Dynamic Programming]
categories: Leetcode
---

这周的比赛还是挺难的。

<!--more-->

## [1089. Duplicate Zeros](https://leetcode.com/problems/duplicate-zeros/description/)

标记难度：Easy

提交次数：1/3

代码效率：30.21%（28ms）

### 题意

给定一个数组，把其中的0都在原位增加成两个，然后把数组截取到原来的长度。问能否在`O(1)`空间复杂度内完成这一操作。

### 分析

如果没有空间复杂度的要求，这道题非常简单：直接新开一个数组存变换后的数组，然后再截回去。但现在显然不能这么做。如何不增加额外空间呢？在0增加而数组长度不变的要求下，数组中总的有效元素数量实际上减少了，也就是原来位于数组尾部的那些元素被丢掉了；只需要把将会留下来的元素按照一定顺序放到空出来的位置即可。在下面这个例子中，将要被丢掉的两个元素是5和0。4原来的位置上将要放0，但是不能直接把0放过去，因为会覆盖掉4；所以应该从右向左逐步向右边空出来的位置上移动。

```txt
        1 0 2 3 0 4 5 0
        1 0 0 2 3 0 0 4 5 0 0

move 4: 1 0 2 3 0 ? 5 4
move 0: 1 0 2 3 ? 0 0 4
move 3: 1 0 2 ? 3 0 0 4
move 2: 1 0 ? 2 3 0 0 4
move 0: 1 0 0 2 3 0 0 4
move 1: 1 0 0 2 3 0 0 4
```

为了找到正确的移动位置，只需知道当前的数字前面有多少个0，所以我们在代码中先从前往后扫描一遍计算出0的总数，再从后往前，一遍移动元素，一边更新前面还有多少个0。时间复杂度是`O(N)`。

不过这么一来，难度怎么看也不像Easy了……

### 代码

```cpp
class Solution {
public:
    void duplicateZeros(vector<int>& arr) {
        int zeroCnt = 0;
        int n = arr.size(), i;
        for (i = 0; i < n; i++) {
            if (arr[i] != 0 && i + zeroCnt >= n)
                break;
            else if (i + zeroCnt + 1 >= n)
                break;
            if (arr[i] == 0)
                zeroCnt++;
        }
        for (; i >= 0; i--) {
            if (arr[i] != 0 && i + zeroCnt < n)
                arr[i + zeroCnt] = arr[i];
            else {
                if (i + zeroCnt < n)
                    arr[i + zeroCnt] = 0;
                if (i + zeroCnt + 1 < n)
                    arr[i + zeroCnt + 1] = 0;
            }
            if (i > 0 && arr[i - 1] == 0)
                zeroCnt--;
        }
    }
};
```

## [1090. Largest Values From Labels](https://leetcode.com/problems/largest-values-from-labels/description/)

标记难度：Medium

提交次数：1/1

代码效率：27.26%（48ms）

### 题意

给定一个集合，其中的每个元素都有一个标签，要求从中选出一个子集，满足子集大小`<=num_wanted`，每种标签对应的元素总数`<=use_limit`，且子集的和最大。

### 分析

这道题倒是很简单的数据结构应用题。直接从每个标签对应的元素中选出前`use_limit`大的元素，再从这些元素里选出前`num_wanted`即可。

### 代码

```cpp
class Solution {
public:
    int largestValsFromLabels(vector<int>& values, vector<int>& labels, int num_wanted, int use_limit) {
        map<int, vector<int>> labelMap;
        int n = values.size();
        for (int i = 0; i < n; i++)
            labelMap[labels[i]].push_back(values[i]);
        // 当时大概脑抽了才用了优先队列
        // 实际上排序就够了
        priority_queue<int> pq;
        for (auto& p: labelMap) {
            sort(p.second.begin(), p.second.end());
            int m = p.second.size();
            for (int i = 0; i < use_limit && i < m; i++)
                pq.push(p.second[m - i - 1]);
        }
        int ans = 0;
        for (int i = 0; i < num_wanted && !pq.empty(); i++) {
            ans += pq.top();
            pq.pop();
        }
        return ans;
    }
};
```

## [1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/description/)

标记难度：Medium

提交次数：1/1

代码效率：40.84%（68ms）

### 题意

求一个八连通方格图从左上角到右下角的最短路，图中有若干障碍物。

### 分析

直接BFS即可。

（本次比赛的前三题难度似乎是递减的。）

### 代码

```cpp
class Solution {
public:
    int shortestPathBinaryMatrix(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        int mx[8] = {1, 1,  1, -1, -1, -1, 0,  0};
        int my[8] = {0, 1, -1,  0,  1, -1, 1, -1};
        vector<vector<bool>> visited(n, vector<bool>(m, 0));
        vector<vector<int>> dist(n, vector<int>(m, -1));
        queue<pair<int, int>> q;
        
        if (grid[0][0] == 1) return -1;
        visited[0][0] = true;
        q.emplace(0, 0);
        dist[0][0] = 1;
        
        while (!q.empty()) {
            int x = q.front().first, y = q.front().second;
            q.pop();
            if (x == n - 1 && y == m - 1)
                return dist[x][y];
            for (int i = 0; i < 8; i++) {
                int nx = x + mx[i], ny = y + my[i];
                if (0 <= nx && nx < n && 0 <= ny && ny < m && !visited[nx][ny] && grid[nx][ny] == 0) {
                    visited[nx][ny] = true;
                    dist[nx][ny] = dist[x][y] + 1;
                    q.emplace(nx, ny);
                }
            }
        }
        
        return -1;
    }
};
```

## [1092. Shortest Common Supersequence](https://leetcode.com/problems/shortest-common-supersequence/description/)

标记难度：Hard

提交次数：2/6

代码效率：

* 普通的写法：35.22%（52ms）
* 更好的写法：0.00%（624ms）

### 题意

给定两个字符串，返回任意一个最短的同时以这两个字符串为子串的字符串。

### 分析

这道题本质上是要求两个字符串的最长公共子串，求出来之后，对于公共子串的每两个字符，把两个字符串中这两个字符之间的字符都塞到公共子串里面去就行了。问题在于如何求最长公共子串。

一种比较省空间的方法是，记录动态规划过程中每一步的计算过程，然后在得到最优解之后回溯。这样做的缺点是比较难写……至少我是这样觉得的，我似乎写了很久。而一种更容易写的方法是，直接在动态规划过程中记录当前的最长公共子串。[^lee]

[^lee]: [lee215's Solution for Leetcode 1092 - \[C++/Python\] Find the LCS](https://leetcode.com/problems/shortest-common-supersequence/discuss/312710/C++Python-Find-the-LCS)

### 代码

#### 普通的写法

```cpp
class Solution {
public:
    string shortestCommonSupersequence(string str1, string str2) {
        int f[1005][1005];
        int last[1005][1005][2];
        int n = str1.length(), m = str2.length();
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                f[i][j] = 0;
                last[i][j][0] = last[i][j][1] = 0;
                // 因为是从0开始的，所以比较麻烦
                if (i > 0 && f[i-1][j] >= f[i][j]) {
                    f[i][j] = f[i-1][j];
                    last[i][j][0] = -1, last[i][j][1] = 0;
                }
                if (j > 0 && f[i][j-1] >= f[i][j]) {
                    f[i][j] = f[i][j-1];
                    last[i][j][0] = 0, last[i][j][1] = -1;
                }
                if (str1[i] == str2[j]) {
                    if (1 > f[i][j]) {
                        f[i][j] = 1;
                        last[i][j][0] = -1, last[i][j][1] = -1;
                    }
                    if (i > 0 && j > 0 && f[i-1][j-1] + 1 >= f[i][j]) {
                        f[i][j] = f[i-1][j-1] + 1;
                        last[i][j][0] = -1, last[i][j][1] = -1;
                    }
                }
            }
        }
        string ans;
        int i = n - 1, j = m - 1;
        while (i >= 0 && j >= 0) {
            while (i >= 0 && j >= 0 && !(last[i][j][0] == -1 && last[i][j][1] == -1) && !(last[i][j][0] == 0 && last[i][j][1] == 0)) {
                if (last[i][j][0] == -1)
                    ans = str1[i] + ans;
                else
                    ans = str2[j] + ans;
                
                int t = last[i][j][1];
                i += last[i][j][0], j += t;
                
            }
            if (i == 0 && j == 0 && last[i][j][0] == 0 && last[i][j][1] == 0)
                break;
            
            if (i >= 0 && j >= 0 && last[i][j][0] == -1 && last[i][j][1] == -1) {
                ans = str1[i] + ans;
                i--;
                j--;
            }
        }
        
        if (i >= 0) ans = str1.substr(0, i + 1) + ans;
        if (j >= 0) ans = str2.substr(0, j + 1) + ans;
        
        return ans;
    }
};
```

#### 更好的写法

大概因为字符串处理太多，这种方法慢得可以。

```cpp
class Solution {
public:
    string shortestCommonSupersequence(string str1, string str2) {
        string f[1001][1001];
        int n = str1.length(), m = str2.length();
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                // 这是一种解决index问题的方法
                if (str1[i] == str2[j])
                    f[i+1][j+1] = f[i][j] + str1[i];
                else
                    f[i+1][j+1] = f[i][j+1].length() > f[i+1][j].length() ? f[i][j+1] : f[i+1][j];
            }
        }
        string s = f[n][m];
        string ans;
        int i = 0, j = 0, k = 0;
        while (i < n && j < m && k < s.length()) {
            while (str1[i] != s[k] && i < n) {
                ans += str1[i];
                i++;
            }
            while (str2[j] != s[k] && j < m) {
                ans += str2[j];
                j++;
            }
            ans += s[k];
            i++, j++, k++;
        }
        // 原来substr还有单变量的版本……
        // http://www.cplusplus.com/reference/string/string/substr/
        ans += str1.substr(i) + str2.substr(j);
        return ans;
    }
};
```
