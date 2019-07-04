---
title: Leetcode Weekly Contest 143总结
urlname: leetcode-weekly-contest-143
toc: true
date: 2019-06-30 13:13:52
updated: 2019-07-05 04:30:00
tags: [Leetcode, Leetcode Contest, alg:Tree, alg:Dynamic Programming, alg:Stack]
categories: Leetcode
---

这周的比赛倒是很简单。

<!--more-->

## [1103. Distribute Candies to People](https://leetcode.com/problems/distribute-candies-to-people/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms

### 题意

把`candies`颗糖分给`n`个人，依次分1，2，3……颗糖，从第1个人分到第n个人后，如果还没分完，继续从第一个人开始。问最后每个人各分了多少颗糖。

### 分析

这道题的思路很简单：首先计算出最多能完整地发几次糖，然后就可以列出公式计算每个人能得到的糖数了。当然，因为数据范围很小，另一种思路是直接暴力。

### 代码

不过，既然我都决定要暴力了，干吗还要算`n`……

```cpp
class Solution {
public:
    vector<int> distributeCandies(int candies, int num_people) {
        int n = sqrt(2 * candies);
        while (n * (n + 1) / 2 <= candies) n++;
        n--;
        vector<int> ans(num_people, 0);
        for (int i = 1; i <= n; i++) {
            ans[(i - 1) % num_people] += i;
            candies -= i;
        }
        ans[n % num_people] += candies;
        return ans;
    }
};
```

## [1104. Path In Zigzag Labelled Binary Tree](https://leetcode.com/problems/path-in-zigzag-labelled-binary-tree/description/)

标记难度：Easy

提交次数：2/3

代码效率：

* 暴力方法：28ms
* 普通方法：0ms

### 题意

给定一棵满二叉树，其中深度为奇数的层结点编号从左到右，深度为偶数的层结点编号从右到左，给定`label`，问根结点到该结点路径上经过的结点。

```txt
     1
   /   \
  3     2
 / \   / \
4   5 6   7
    ...
```

### 分析

最简单的思路是干脆把这棵树先建出来，然后模拟从`label`走到根的过程。比赛的时候我看到这道题一时心慌（感觉不大做得出来），于是就写了这种傻瓜做法，空间和时间复杂度都是`O(N)`。

当然，这种在树上走的题一般都可以做到时间复杂度`O(log(N))`（虽然此处没有实际的树）。虽然`label`在树中的位置相比正常的满二叉树发生了变化，不过它对应的层并没有变，所以可以首先计算出当前的层数，然后再换算每次上一层的label是什么。通过观察可以发现，无论本层是奇数层还是偶数层，公式都是一样的。

### 代码

#### 暴力方法

```cpp
class Solution {
public:
    vector<int> pathInZigZagTree(int label) {
        int tree[1000005];
        memset(tree, -1, sizeof(tree));
        int pow = 1, sum = 1;
        int index = -1;
        for (int i = 1; sum <= label; i++) {
            int last = sum - 1;
            if (i % 2 == 1) {
                for (int j = 1; j <= pow; j++) {
                    tree[sum] = last + j;
                    if (last + j == label) {
                        index = sum;
                        break;
                    }
                    sum++;
                }
            }
            else {
                for (int j = pow; j >= 1; j--) {
                    tree[sum] = last + j;
                    if (last + j == label) {
                        index = sum;
                        break;
                    }
                    sum++;
                }
            }
            if (index != -1) break;
            pow *= 2;
        }
        vector<int> ans;
        while (index > 0) {
            ans.push_back(tree[index]);
            index /= 2;
        }
        reverse(ans.begin(), ans.end());
        return ans;
    }
};
```

#### 普通方法

```cpp
class Solution {
public:
    vector<int> pathInZigZagTree(int label) {
        int level = 1;
        while ((1 << level) <= label) level++;
        vector<int> ans;
        while (level > 0) {
            ans.push_back(label);
            label = (1 << level) - 1 - (label - (1 << (level-1)));
            label /= 2;
            level--;
        }
        reverse(ans.begin(), ans.end());
        return ans;
    }
};
```

## [1105. Filling Bookcase Shelves](https://leetcode.com/problems/filling-bookcase-shelves/description/)

标记难度：Medium

提交次数：1/1

代码效率：28ms

### 题意

有若干本书，要求将这些书按顺序摆放在书架上：每次取出还没有摆完的编号最小的连续的且总宽度不超过书架宽度的几本书，摞到之前的层上，本层高度定为最高的书的高度，问把书摆完之后书架的最低高度。

### 分析

简单的DP题。用`f[i]`表示把第`[0..i]`本书放完时的最小高度，`g[i][j]`表示第`[i..j]`本书的总宽度，`h[i..j]`表示第`[i..j]`本书的最大高度。于是可以得到递推式`f[i] = min{f[j] + h[j+1..i]} (j < i && g[j+1..i] < maxWidth)`，时间复杂度为`O(N^2)`。通过在DP过程中进行DP，`h`和`g`数组都可以省略，详见[Java DP solution](https://leetcode.com/problems/filling-bookcase-shelves/discuss/323315/Java-DP-solution)。

### 代码

```cpp
class Solution {
public:
    int minHeightShelves(vector<vector<int>>& books, int shelf_width) {
        int f[1001];  // f[i]：把前i本书放完时的最小高度
        int g[1001][1001];  // g[i][j]: books[i..j]的宽度（可以用前缀数组代替）
        int h[1001][1001];  // h[i][j]: books[i..j]的最大高度
        int n = books.size(), m = shelf_width;
        // books[i][0]是宽度，books[i][1]是高度
        for (int i = 0; i < n; i++) {
            f[i] = 1e9;
            g[i][i] = books[i][0];
            h[i][i] = books[i][1];
            for (int j = i + 1; j < n; j++) {
                g[i][j] = g[i][j-1] + books[j][0];
                h[i][j] = max(h[i][j-1], books[j][1]);
            }
        }
        f[0] = books[0][1];
        for (int i = 1; i < n; i++) {
            // 一次把[j..i]都放了
            for (int j = i; j >= 0; j--) {
                if (g[j][i] > shelf_width) break;
                int x = j == 0 ? 0 : f[j-1];
                f[i] = min(f[i], x + h[j][i]);
            }
        }
        return f[n-1];
    }
};
```

## [1106. Parsing A Boolean Expression](https://leetcode.com/problems/parsing-a-boolean-expression/description/)

标记难度：Hard

提交次数：1/2

代码效率：28ms

### 题意

给定一个布尔表达式，求值。

### 分析

和上周的[1096. Brace Expansion II](https://leetcode.com/problems/brace-expansion-ii)挺像的，但是要简单一些。直接用一个栈模拟就行了。

### 代码

```cpp
class Solution {
public:
    bool parseBoolExpr(string expression) {
        vector<char> stk;
        for (char c: expression) {
            if (c == ')') {
                vector<bool> a;
                while (!stk.empty() && stk.back() != '(') {
                    if (stk.back() == 't') a.push_back(true);
                    else if (stk.back() == 'f') a.push_back(false);
                    stk.pop_back();
                }
                stk.pop_back(); // (
                char type = stk.back();
                stk.pop_back();
                
                bool f = a[0];
                if (type == '!') f = !a[0];
                else {
                    for (int i = 1; i < a.size(); i++)
                        f = type == '|' ? (f || a[i]) : (f && a[i]);
                }
                stk.push_back(f == true ? 't' : 'f');
            }
            else
                stk.push_back(c);
        }
        char ans = stk.back();
        return ans == 't' ? true : false;
    }
};
```
