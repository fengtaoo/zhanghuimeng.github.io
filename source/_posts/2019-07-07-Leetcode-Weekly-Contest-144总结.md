---
title: Leetcode Weekly Contest 144总结
urlname: leetcode-weekly-contest-144
toc: true
date: 2019-07-07 20:57:44
updated: 2019-07-07 22:18:00
tags: [Leetcode, Leetcode Contest, alg:String, alg:Binary Tree, alg:Greedy]
categories: Leetcode
---

这次的题比较水，甚至都没有Hard的。

<!--more-->

## [1108. Defanging an IP Address](https://leetcode.com/problems/defanging-an-ip-address/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms

### 题意

把一个IPv4地址中的每个点都换成`[.]`。

### 分析

这道题实在是过于简单了。我觉得对于C++来说，最好的方法还是直接把字符串过一遍，但如果是Python之类的话，估计正则或者replace更直接。

### 代码

```cpp
class Solution {
public:
    string defangIPaddr(string address) {
        string ans;
        for (char c: address) {
            if (c == '.') ans += "[.]";
            else ans += c;
        }
        return ans;
    }
};
```

## [1109. Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/description/)

标记难度：Medium

提交次数：1/1

代码效率：77.41%（268ms）

### 题意

有`n`条线段，每条线段有起止位置和权重，问所有线段加起来之后线上每个点的值。

### 分析

看起来像是线段树，不过只有加减操作，没有单次查询操作时其实并不需要线段树，用一个delta数组记录进入当前点时权值的变化就可以了。

### 代码

```cpp
class Solution {
public:
    vector<int> corpFlightBookings(vector<vector<int>>& bookings, int n) {
        int delta[20005];
        memset(delta, 0, sizeof(delta));
        for (int i = 0; i < bookings.size(); i++) {
            delta[bookings[i][0]] += bookings[i][2];
            delta[bookings[i][1] + 1] -= bookings[i][2];
        }
        vector<int> ans;
        int cur = 0;
        for (int i = 1; i <= n; i++) {
            cur += delta[i];
            ans.push_back(cur);
        }
        return ans;
    }
};
```

## [1110. Delete Nodes And Return Forest](https://leetcode.com/problems/delete-nodes-and-return-forest/description/)

标记难度：Medium

提交次数：1/1

代码效率：92.36%（20ms）

### 题意

给定一棵二叉树和一些要删除的结点，返回剩下的森林中的所有树的根结点。

### 分析

大体来说，一边DFS一边删就可以了。先处理左右子结点，然后检查当前结点是否需要被删除，如果是，则将左右子结点（如果存在）添加到森林的根列表中。同时还需要维护子结点指针的变化（因为有可能被删掉）。

### 代码

```cpp
class Solution {
    vector<TreeNode*> ans;
    
    TreeNode* dfs(TreeNode* root, set<int>& delSet) {
        if (root == NULL) return NULL;
        root->left = dfs(root->left, delSet);
        root->right = dfs(root->right, delSet);
        if (delSet.find(root->val) != delSet.end()) {
            if (root->left != NULL) ans.push_back(root->left);
            if (root->right != NULL) ans.push_back(root->right);
            return NULL;
        }
        return root;
    }
    
public:
    vector<TreeNode*> delNodes(TreeNode* root, vector<int>& to_delete) {
        set<int> delSet;
        for (int x: to_delete) delSet.insert(x);
        root = dfs(root, delSet);
        if (root != NULL) ans.push_back(root);
        return ans;
    }
};
```

## [1111. Maximum Nesting Depth of Two Valid Parentheses Strings](https://leetcode.com/problems/maximum-nesting-depth-of-two-valid-parentheses-strings/description/)

标记难度：Medium

提交次数：1/1

代码效率：100.00%（8ms）

### 题意

给定一个括号序列，需要把它分成两个子序列，使得这两个子序列的深度相差最少。

### 分析

最开始看到这题，我一脸懵逼，不知道该怎么做；后来看了[别人的做法](https://leetcode.com/problems/maximum-nesting-depth-of-two-valid-parentheses-strings/discuss/328841/JavaC++Python-Several-Ideas)，发现是非常简单的贪心。思路就是遍历这个括号序列，然后看情况把当前的括号分给其中一个子序列。为这两个子序列都维护当前深度（也就是还没有关闭的左括号的数目），使得这个深度尽量相等。至于为啥……啊，这个好难解释啊，不如意会一下吧……

### 代码

```cpp
class Solution {
public:
    vector<int> maxDepthAfterSplit(string seq) {
        vector<int> ans;
        int cnt[2], depth[2];
        string s[2];
        cnt[0] = cnt[1] = 0;
        depth[0] = depth[1] = 0;
        // 保持平均
        for (char c: seq) {
            if (c == '(') {
                if (cnt[0] < cnt[1]) {
                    cnt[0]++;
                    ans.push_back(0);
                    s[0] += c;
                }
                else {
                    cnt[1]++;
                    ans.push_back(1);
                    s[1] += c;
                }
            }
            else {
                if (cnt[0] > cnt[1]) {
                    cnt[0]--;
                    ans.push_back(0);
                    s[0] += c;
                }
                else {
                    cnt[1]--;
                    ans.push_back(1);
                    s[1] += c;
                }
            }
            depth[0] = max(depth[0], cnt[0]);
            depth[1] = max(depth[1], cnt[1]);
        }
        return ans;
    }
};
```
