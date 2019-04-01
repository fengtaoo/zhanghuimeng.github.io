---
title: Leetcode Weekly Contest 127总结
urlname: leetcode-weekly-contest-127
toc: true
date: 2019-03-13 23:38:58
updated: 2019-03-31 15:32:00
tags: [Leetcode, Leetcode Contest, alg:Binary Search Tree]
categories: Leetcode
---

这周的题目都很简单，没有特别难的。

<!--more-->

## [1005. Maximize Sum Of Array After K Negations](https://leetcode.com/problems/maximize-sum-of-array-after-k-negations/description/)

标记难度：Easy

提交次数：1/1

代码效率：97.24%（8ms）

### 题意

给定一个数组，将其中的数改变正负`k`次，问能得到的最大和是多少？

### 分析

水题。首先按绝对值从小到大把所有负数都变成正的，然后一直变绝对值最小的数到次数耗尽即可。

### 代码

```cpp
class Solution {
public:
    int largestSumAfterKNegations(vector<int>& A, int K) {
        sort(A.begin(), A.end());
        int minAbs = 10000, idx = -1;
        for (int i = 0; i < A.size(); i++) {
            if (K <= 0) continue;
            else {
                if (A[i] < 0) {
                    A[i] = -A[i];
                    K--;
                }
            }
            if (abs(A[i]) < minAbs)
                minAbs = abs(A[i]), idx = i;
        }
        while (K > 0) {
            A[idx] = - A[idx];
            K--;
        }
        int sum = 0;
        for (int x: A)
            sum += x;
        return sum;
    }
};
```

## [1006. Clumsy Factorial](https://leetcode.com/problems/clumsy-factorial/description/)

标记难度：Easy

提交次数：1/1

代码效率：100.00%（4ms）

### 题意

给定一个数组，将其中的数改变正负`k`次，问能得到的最大和是多少？

### 分析

同样是水题。按四个数一组的规律模拟即可。

### 代码

```cpp
class Solution {
public:
    int clumsy(int N) {
        long long int ans = 0, cur = 0;
        for (int i = N; i >= 1; i -= 4) {
            cur = i;
            if (i > 1) cur *= i - 1;
            if (i > 2) cur /= i - 2;
            if (i == N) ans += cur;
            else ans -= cur;
            if (i > 3) ans += i - 3;
        }
        return ans;
    }
};
```

## [1007. Minimum Domino Rotations For Equal Row](https://leetcode.com/problems/minimum-domino-rotations-for-equal-row/description/)

标记难度：Medium

提交次数：1/1

代码效率：0.00%（252ms）

### 题意

有一排多米诺骨牌，这些牌上半部分写了一个数字（`A[i]`），下半部分写了另一个数字（`B[i]`）。问是否有一种旋转某些骨牌的方法，使得所有牌的上半部分或下半部分的数字都是相同的，且旋转总次数最少。

### 分析

还是水题。首先统计每个数字在哪些骨牌上出现了；如果不存在出现在所有骨牌上的数字，则显然不存在相应的方法。然后对于每个数字，如果它在所有骨牌上都出现了，则分别尝试把该数字旋转到上半部分或下半部分，统计旋转次数，取最小值。

显然，在所有骨牌上都出现的数字最多只可能有两个，所以算法的复杂度是比较低的（虽然我写得不是很低）。

### 代码

```cpp
class Solution {
public:
    int minDominoRotations(vector<int>& A, vector<int>& B) {
        set<int> numbers[7];  // 显然，其实可以不用set的
        int n = A.size();
        for (int i = 0; i < n; i++) {
            numbers[A[i]].insert(i);
            numbers[B[i]].insert(i);
        }
        int ans = -1;
        // 即使数字很多，本质上也是O(n)的
        for (int i = 1; i <= 6; i++) {
            if (numbers[i].size() == n) {
                int cnt = 0;
                // flip up
                for (int j = 0; j < n; j++)
                    if (A[j] != i) cnt++;
                ans = ans == -1 ? cnt : min(cnt, ans);
                // flip down
                cnt = 0;
                for (int j = 0; j < n; j++)
                    if (B[j] != i) cnt++;
                ans = ans == -1 ? cnt : min(cnt, ans);
            }
        }
        return ans;
    }
};
```

## [1008. Construct Binary Search Tree from Preorder Traversal](https://leetcode.com/problems/construct-binary-search-tree-from-preorder-traversal/description/)

标记难度：Medium

提交次数：1/2

代码效率：20.38%（12ms）

### 题意

给定一棵**二叉搜索树**的先序遍历，求该二叉搜索树。

### 分析

显然，如果只给一个先序遍历，那么不可能恢复二叉树，因为无法把左子树和右子树分开。但是，有了**二叉搜索树**这个条件之后，很显然，先序遍历序列中比根小的元素属于左子树，比根大的元素属于右子树（所有元素两两不等）。然后就可以递归了。

### 代码

```cpp
class Solution {
private:
    TreeNode* dfs(vector<int>& preorder, int l, int r) {
        if (l > r) return NULL;
        if (l == r) return new TreeNode(preorder[l]);
        int x = preorder[l];
        int m = l + 1;
        while (m <= r && preorder[m] <= x) m++;
        TreeNode* root = new TreeNode(x);
        root->left = dfs(preorder, l + 1, m - 1);
        root->right = dfs(preorder, m, r);
        return root;
    }
    
public:
    TreeNode* bstFromPreorder(vector<int>& preorder) {
        return dfs(preorder, 0, preorder.size() - 1);
    }
};
```
