---
title: Leetcode Weely Contest 131总结
urlname: leetcode-weekly-contest-131
toc: true
date: 2019-04-07 14:15:52
updated: 2019-04-07 14:15:52
tags: [Leetcode, Leetcode Contest, alg:Depth-first Search, alg:Breadth-first Search, alg:Dynamic Programming, alg:Greedy]
categories: Leetcode
---

这周的比赛十分简单。

<!--more-->

## [1021. Remove Outermost Parentheses](https://leetcode.com/problems/remove-outermost-parentheses/description/)

标记难度：Easy

提交次数：1/1

代码效率：12ms

### 题意

给定一个合法括号序列，把它切成尽可能多的合法括号序列，去掉每个序列的最外层括号，最后再拼起来。

### 分析

直接用栈或者类似的方法搞一下就可以了。比如说，只记录open parentheses数量（或者说栈中元素数量）也算是一个不错的方法……[^lee]

[^lee]: [lee215's Solution for Leetcode 1021 - \[Java/C++/Python\] Count Opened Parenthesis](https://leetcode.com/problems/remove-outermost-parentheses/discuss/270022/JavaC++Python-Count-Opened-Parenthesis)

### 代码

```cpp
class Solution {
public:
    string removeOuterParentheses(string S) {
        stack<char> stk;
        string t, ans;
        for (char c: S) {
            if (c == ')' && !stk.empty() && stk.top() == '(') stk.pop();
            else stk.push(c);
            t += c;
            if (stk.size() == 0) {
                ans += t.substr(1, t.length() - 2);
                t = "";
            }
        }
        return ans;
    }
};
```

## [1022. Sum of Root To Leaf Binary Numbers](https://leetcode.com/problems/sum-of-root-to-leaf-binary-numbers/description/)

标记难度：Easy

提交次数：1/2

代码效率：12ms

### 题意

给定一棵二叉树，树上每个结点都有一个0或1的值，每条从根到叶子的路径都表示一个二进制数，其中根是高位，叶子是低位。问树中所有二进制数的和。

### 分析

反正树上叶子的数量不会超过结点总数，所以不需要特别的优化，做DFS或BFS然后求和即可。不过，这道题的数据范围只有1000；如果结点总数很多，在树变成一条链的状况下递归DFS会爆栈，所以BFS更好一些。

（不过我懒得写了。）

### 代码

```cpp
typedef long long int LL;
class Solution {
private:
    LL sum;
    LL mod;
    
    void dfs(TreeNode* root, LL x) {
        if (root == NULL) return;
        if (root->left == NULL && root->right == NULL) {
            sum = (sum + x) % mod;
            return;
        }
        if (root->left != NULL) dfs(root->left, x * 2 + root->left->val);
        if (root->right != NULL) dfs(root->right, x * 2 + root->right->val);
    }
    
public:
    int sumRootToLeaf(TreeNode* root) {
        sum = 0;
        mod = 1e9 + 7;
        dfs(root, root->val);
        return sum;
    }
};
```

## [1023. Camelcase Matching](https://leetcode.com/problems/camelcase-matching/description/)

标记难度：Medium

提交次数：2/3

代码效率：

* 复杂的做法：8ms
* 简单的做法：8ms

### 题意

有两个驼峰写法的字符串，问能否在`query`字符串中插入若干个**小写字母**，使得它与`pattern`串相等？

### 分析

这个问题实际上是两个问题的结合体。首先，如果`query`能变成和`pattern`相等，那么它们内部含有的大写字母集合应该是相等的；除此之外，对于每两个大写字母之间的字符串，`query`串都是`pattern`串的子序列。于是这个问题就被降维了。

比赛的时候我写的判断子序列方法是DP，后来一想，发现显然贪心也能做这个。用所谓“two pointer”的方法：

* `i`指向`query`下一个尚未被匹配的字母，`j`指向`pattern`下一个尚未被查看的字母
* 如果`query[i] == query[j]`，`i++`，`j++`；否则`j++`
* 如果`query`先耗尽，则它是`pattern`的子序列；否则不是

贪心的正确性是显然的。

最后想了想，其实根本就不需要把这个问题降维，只要做一个有特判的贪心判断子序列就行了。[^j2tp]

[^j2tp]: [yuanb10's Solution for Leetcode 1023 - Java Easy Two Pointers](https://leetcode.com/problems/camelcase-matching/discuss/270006/Java-Easy-Two-Pointers)

吐槽：看测试样例，这道题如果是面试题，肯定是FB出的。

### 代码

#### 复杂的代码

```cpp
class Solution {
private:
    // 其实没什么用的判断
    string getCamel(string s) {
        string x;
        for (char c: s)
            if ('A' <= c && c <= 'Z')
                x += c;
        return x;
    }

    // 这个函数完全可以取代上面那个
    vector<string> camelSplit(string s) {
        vector<string> v;
        string x;
        for (char c: s) {
            if ('A' <= c && c <= 'Z') {
                v.push_back(x);
                x = "";
            }
            else
                x += c;
        }
        v.push_back(x);
        return v;
    }
    
    bool f[105][105];
    
    // 复杂的判断子序列的方法
    bool isIn(string q, string a) {
        if (q.length() > a.length()) return false;
        if (q.length() == 0) return true;
        int n = q.length(), m = a.length();
        memset(f, 0, sizeof(f));
        for (int i = 0; i < n; i++) {
            for (int j = i; j < m; j++) {
                if (i == 0) {
                    if (j == 0) f[i][j] = q[i] == a[j];
                    else f[i][j] = f[i][j-1] | (q[i] == a[j]);
                    continue;
                }
                // q[0..i] in a[0..j]
                if (q[i] == a[j]) f[i][j] |= f[i-1][j-1];
                f[i][j] |= f[i][j-1];
            }
        }
        return f[n-1][m-1];
    }
    
public:
    vector<bool> camelMatch(vector<string>& queries, string pattern) {
        string camel = getCamel(pattern);
        vector<string> sp(camelSplit(pattern));
        vector<bool> ans;
        for (string query: queries) {
            string c1 = getCamel(query);
            // 如果大写字母集合不同，则显然失败
            if (c1 != camel) {
                ans.push_back(false);
                continue;
            }
            vector<string> sp2(camelSplit(query));

            // 判断每个间隔中的子序列
            bool ok = true;
            for (int i = 0; i < sp.size(); i++) {
                if (!isIn(sp[i], sp2[i])) {
                    ok = false;
                    break;
                }
            }
            ans.push_back(ok);
        }
        return ans;
    }
};
```

#### 简单的代码

```cpp
class Solution {
public:
    vector<bool> camelMatch(vector<string>& queries, string pattern) {
        vector<bool> ans;
        for (string query: queries) {
            int n = query.length(), m = pattern.length();
            int j = 0;
            bool ok = true;
            for (int i = 0; i < n; i++) {
                // 消耗pattern中的小写字母
                if (j < m && query[i] == pattern[j]) j++;
                // query中多出来了大写字母
                else if ('A' <= query[i] && query[i] <= 'Z') {
                    ok = false;
                    break;
                }
            }
            // pattern没被消耗完
            if (!ok || j != m) ans.push_back(false);
            else ans.push_back(true);
        }
        return ans;
    }
};
```

## [1024. Video Stitching](https://leetcode.com/problems/video-stitching/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* DP：12ms
* 贪心：4ms

### 题意

给定一系列子区间，问覆盖`[0, T)`区间最少需要几个子区间。

### 分析

我最开始看错题了，以为是让我们求这些Video粘成一整个最少需要剪几刀。后来定睛一看，发现是最小覆盖问题，然后也就继续DP下去了。

DP大概是这么个思路：`f[l][r]`表示覆盖`[l, r)`区间最少需要几个子区间，然后对于所有的子区间，判断它和`[l, r)`是否有交集，如果有的话，就计算覆盖去掉交集的这个部分最少需要几个子区间。

复杂度有些难以分析，也许是`O(T^2 * N)`吧。

当然，这道题完全可以贪心。贪心的思路是这样的：首先找出起始点为`0`的所有子区间中末尾最远的区间，然后在这个区间中寻找末尾更远的区间，以此类推。当然，看起来需要预处理一个起始点在`[0, end]`范围内的区间的最远的末尾的数组……或者也可以不这么写。[^note]

[^note]: [leetcode 5019 Video Stitching解题笔记](http://www.noteanddata.com/leetcode-5019-Video-Stitching-java-solution-note.html)

---

![9102年的5019题](5019.png)

吐槽：Leetcode怎么都有5019题了，不愧是9102年。（现在这个bug已经修好了）

### 代码

#### DP

```cpp
class Solution {
private:
    int f[105][105];
    int calc(int l, int r, vector<vector<int>>& clips) {
        if (l >= r) {
            f[l][r] = 0;
            return 0;
        }
        if (f[l][r] != -1)
            return f[l][r];
        f[l][r] = 1e9;
        for (int i = 0; i < clips.size(); i++) {
            if (clips[i][1] <= l || r <= clips[i][0]) continue;
            int x = 1;
            if (l < clips[i][0]) x += calc(l, clips[i][0], clips);
            if (clips[i][1] < r) x += calc(clips[i][1], r, clips);
            f[l][r] = min(f[l][r], x);
        }
        return f[l][r];
    }
    
public:
    int videoStitching(vector<vector<int>>& clips, int T) {
        memset(f, -1, sizeof(f));
        int ans = calc(0, T, clips);
        return ans == 1e9 ? -1 : ans;
    }
};
```

#### 贪心

```cpp
class Solution {
public:
    int videoStitching(vector<vector<int>>& clips, int T) {
        sort(clips.begin(), clips.end());
        int maxEnd[105];
        memset(maxEnd, 0, sizeof(maxEnd));
        int n = clips.size();
        // 预处理数组
        int cur = 0;
        maxEnd[0] = -1;
        for (int i = 0; i < n; i++) {
            if (clips[i][0] > cur) {
                if (maxEnd[cur] == -1) return -1;
                while (cur < clips[i][0]) {
                    maxEnd[cur + 1] = maxEnd[cur];
                    cur++;
                }
            }
            maxEnd[cur] = max(maxEnd[cur], clips[i][1]);
        }
        
        for (int i = 1; i <= T; i++)
            maxEnd[i] = max(maxEnd[i-1], maxEnd[i]);
        
        int ans = 0;
        cur = 0;
        while (cur < T) {
            if (cur >= maxEnd[cur]) return -1;
            cur = maxEnd[cur];
            ans++;
        }
        return ans;
    }
};
```