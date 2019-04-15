---
title: Leetcode Weekly Contest 132总结
urlname: leetcode-weekly-contest-132
toc: true
date: 2019-04-14 13:21:47
updated: 2019-04-14 13:21:47
tags: [Leetcode, Leetcode Contest]
categories: Leetcode
---

这周的比赛还是不太难，重点是我认识到了`unordered_map`相比`map`在速度上是有优势的（换言之，`unordered_map`不是彻头彻尾的鸡肋）。

<!--more-->

## [1025. Divisor Game](https://leetcode.com/problems/divisor-game/description/)

标记难度：Easy

提交次数：2/3

代码效率：

* 没推导：4ms
* 推导了：4ms

### 题意

两个人（Alice和Bob）轮流玩一个游戏：

* 黑板上有一个数`N`
* 每一步可以任选满足`0 < x < N`且`N % x == 0`的`x`，将`N`换成`N - x`
* 当`N = 1`时当前玩家就输了

给定`N`，问当前玩家（Alice）能否赢？

### 分析

看到题目之后，我很自然地写了个极大极小搜索，调了调就交上去了。当然，这没有什么问题，但实在有一些愚蠢（如果不是极其愚蠢的话；这种行为是在用敲代码代替思考，而且为什么要在Leetcode比赛的第一题使用这种东西。。）。实际上可以直接推出来获胜条件的。

如果`N = 1`，显然Alice输了。

如果`N = 2`，则Alice可以（只能）选择`x = 1`，之后`N = 1`（Bob），Bob输了，Alice赢了。

如果`N = 3`，则Alice仍然只能选择`x = 1`，之后`N = 2`（Bob），Bob必胜，Alice输了。

如果`N = 4`，此时Alice有两种选择，`x = 1`或者`x = 2`：

* 如果Alice选择`x = 1`，接下来的局面就是`N = 3`（Bob），Bob必输，Alice获胜。
* 如果Alice选择`x = 2`，接下来的局面就是`N = 2`（Bob），Bob必胜，Alice输了。

因为Alice有至少一种选择可以获胜，所以`N = 4`局面对Alice是必胜的。

如果`N = 5`，此时Alice只能选择`x = 1`，之后`x = 4`（Bob），Bob必胜，Alice输了。

如果`N = 6`，此时Alice有3种选择，`x = 1`，`x = 2`或者`x = 3`：

* 如果Alice选择`x = 1`，接下来的局面就是`N = 5`（Bob），Bob必输，Alice获胜。
* 如果Alice选择`x = 2`，接下来的局面就是`N = 4`（Bob），Bob必胜，Alice输了。
* 如果Alice选择`x = 3`，接下来的局面就是`N = 3`（Bob），Bob必输，Alice获胜。

因为Alice有至少一种选择可以获胜，所以`N = 6`局面对Alice是必胜的。

。。。。。。

---

虽然上面只是手动做了一堆极大极小过程的推导，不过还是很容易看出，好像Alice在`N`为偶数时必胜，在`N`为奇数时必输。下面用数学归纳法证明这个结论。

初始条件：上面已经说明，`N = 1`时Alice胜利，`N = 2`时Alice失败。

假设：`1 <= N <= 2*n (n >= 1)`时，结论成立。

下证结论对`N = 2*n + 1`和`N = 2*n + 2`成立。`N = 2*n + 1`时，显然`N`的任何因子`x`都是奇数，因此`N - x`为偶数：因此，对任何局面`N - x`（Bob），Bob都必胜，因此Alice必输。而`N = 2*n + 2`时，Alice可以选择`x = 1`，此时对于局面`N - 1`（Bob），Bob必输，因此Alice获胜。

### 代码

#### min-max搜索

```cpp
class Solution {
private:
    int win[1005] = {-1};
    
public:
    bool f(int N) {
        if (win[N] != -1)
            return win[N];
        if (N == 1) {
            win[N] = false;
            return win[N];
        }
        for (int x = 1; x < N; x++) {
            if (N % x == 0 && !f(N - x)) {
                win[N] = true;
                return true;
            }
        }
        win[N] = false;
        return false;
    }
    
    bool divisorGame(int N) {
        memset(win, -1, sizeof(win));
        return f(N);
    }
};
```

#### 还是min-max（直接用推导结论）

```cpp
class Solution {
public:
    bool divisorGame(int N) {
        return N % 2 == 0;
    }
};
```

## [1026. Maximum Difference Between Node and Ancestor](https://leetcode.com/problems/maximum-difference-between-node-and-ancestor/description/)

标记难度：Easy

提交次数：1/2

代码效率：12ms

### 题意

求一棵二叉树上所有祖先结点和后代结点的值的差的绝对值的最大值。

### 分析

大水题，直接做个BFS/DFS，记录直接祖先结点的最大值或者最小值，和当前结点值相减然后更新答案就行。

### 代码

```cpp
class Solution {
public:
    int maxAncestorDiff(TreeNode* root) {
        queue<pair<TreeNode*, pair<int, int>>> q;  // node, minVal, maxVal;
        int ans = 0;
        q.emplace(root, make_pair(1e9, -1e9));
        while (!q.empty()) {
            TreeNode* p = q.front().first;
            int minn = q.front().second.first;
            int maxn = q.front().second.second;
            q.pop();
            minn = min(minn, p->val);
            maxn = max(maxn, p->val);
            ans = max(ans, abs(minn - p->val));
            ans = max(ans, abs(maxn - p->val));
            if (p->left != NULL)
                q.emplace(p->left, make_pair(minn, maxn));
            if (p->right != NULL)
                q.emplace(p->right, make_pair(minn, maxn));
        }
        return ans;
    }
};
```

## [1027. Longest Arithmetic Sequence](https://leetcode.com/problems/longest-arithmetic-sequence/description/)

标记难度：Medium

提交次数：3/3

代码效率：

* 愚蠢的map：3064ms
* 愚蠢的unordered_map：2148ms

### 题意

给定数组`A`，求`A`中最长的等差数列的长度。`A.length <= 2000`，`0 <= A[i] <= 10000`。

### 分析

这道题看起来像是[USACO 1.5.1: Arithmetic Progressions（枚举）](/post/usaco-1-5-1-arithmetic-progressions/)，虽然那道题是判断有没有长度大于`N`的等差数列的。我想，这道题应该也有很多优化方法。

#### 愚蠢的map和unordered_map方法

我什么都没想，直接开了2000个map，用来存公差->具有该公差且以`A[i]`为结尾的等差数列的最长长度。然后就3000ms了，然后leetcode居然还给过了。

### 代码

#### 愚蠢的map/unordered_map

```cpp
class Solution {
    map<int, int> gapMap[2005];
    // 换成下面这一行可提速33%
    // unordered_map<int, int> gapMap[2005];
public:
    int longestArithSeqLength(vector<int>& A) {
        int ans = 1;
        int n = A.size();
        for (int i = 0; i < n; i++)
            gapMap[i].clear();
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++) {
                int d = A[i] - A[j];
                int len = gapMap[j][d];
                gapMap[i][d] = len + 1;
                ans = max(len + 1, ans);
            }
        }
        return ans + 1;
    }
};
```

## [1028. Recover a Tree From Preorder Traversal](https://leetcode.com/problems/recover-a-tree-from-preorder-traversal/description/)

标记难度：Hard

提交次数：1/1

代码效率：40ms

### 题意

### 分析

这又是一道“你好好想想为什么leetcode的树序列化输入长那个样子……”系列的题（我是说，在只已知一种树的遍历序列的前提下，显然没法把树复原出来，那么应该加上什么限制条件才能做到这一点？），这次是先序遍历+深度。

说实话，我觉得这是道水题。

（虽然眼下流行的是stack方法，这种方法更好，但也更难想。）

### 代码

```cpp
class Solution {
public:
    TreeNode* dfs(vector<pair<int, int>>& preorder, int l, int r) {
        if (l > r) return NULL;
        TreeNode* root = new TreeNode(preorder[l].first);
        if (l == r) return root;
        // preorder[l+1]是左子树树根
        int depth = preorder[l + 1].second;
        int m;
        // 通过深度找右子树树根
        for (m = l + 2; m <= r; m++)
            if (preorder[m].second == depth)
                break;
        // 重建
        if (m > r)
            root->left = dfs(preorder, l + 1, r);
        else {
            root->left = dfs(preorder, l + 1, m - 1);
            root->right = dfs(preorder, m, r);
        }
        return root;
    }
    
    TreeNode* recoverFromPreorder(string S) {
        vector<pair<int, int>> preorder; // val, depth
        // 所以我首先把值和深度预处理出来了……
        int i = 0, n = S.length();
        while (i < n) {
            int depth = 0;
            while (i < n && S[i] == '-') {
                i++;
                depth++;
            }
            string val;
            while (i < n && S[i] != '-') {
                val += S[i];
                i++;
            }
            preorder.emplace_back(stoi(val), depth);
        }
        return dfs(preorder, 0, preorder.size() - 1);
    }
};
```
