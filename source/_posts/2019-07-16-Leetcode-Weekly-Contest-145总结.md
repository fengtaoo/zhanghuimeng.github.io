---
title: Leetcode Weekly Contest 145总结
urlname: leetcode-weekly-contest-145
toc: true
date: 2019-07-16 00:47:45
updated: 2019-07-23 04:08:00
tags: [Leetcode, Leetcode Contest, alg:Tree, alg:Greedy, alg:Dynamic Programming]
categories: Leetcode
---

这次比赛有（bu）幸遇到了两道我不大确定怎么做的题，所以拖到了现在。当然，也算是学习到了一个新的范式吧。

<!--more-->

## [1122. Relative Sort Array](https://leetcode.com/problems/relative-sort-array/description/)

标记难度：Easy

提交次数：1/1

代码效率：42.90%（8ms）

### 题意

给定两个数组，其中一个数组的值是互异的，另一个数组的值全都包含在此数组中，要求将后一个数组排序，使得元素的出现顺序与第一个数组相符。

### 分析

这道题最好的做法显然是使用哈希表（或map）统计后一个数组中的元素，然后直接把这个数组重新构建出来。

以及我觉得这道题可以加一些follow-up，比如如何进行原地排序，以及能否在附加空间复杂度为`O(1)`的要求下排序。原地排序很简单，后一个感觉很难做到……

### 代码

写的时候直接重新构造了一个数组，不过改成原地的形式应该是非常简单的。

```cpp
class Solution {
public:
    vector<int> relativeSortArray(vector<int>& arr1, vector<int>& arr2) {
        map<int, int> myMap;
        for (int x: arr1) myMap[x]++;
        vector<int> ans;
        for (int x: arr2) {
            for (int i = 0; i < myMap[x]; i++)
                ans.push_back(x);
            myMap.erase(x);
        }
        for (auto& p: myMap) {
            for (int i = 0; i < p.second; i++)
                ans.push_back(p.first);
        }
        return ans;
    }
};
```

## [1123. Lowest Common Ancestor of Deepest Leaves](https://leetcode.com/problems/lowest-common-ancestor-of-deepest-leaves/description/)

标记难度：Medium

提交次数：1/1

代码效率：7.23%（32ms）

### 题意

求出树中所有深度最大的叶子的LCA。

### 分析

这道题的题面比较令人困惑，实际上是这样的：首先树中的所有的叶子有一个最大的深度，然后对于深度和这个值相同的所有叶子，求它们的共同LCA。因为这实在是太困惑了，我干脆摸出了一个LCA模板，直接求出了所有深度最大的叶子，然后用模板求了LCA。当然，因为LCA的原始模板返回的是结点编号，我还专门弄了一个数组来存编号到`TreeNode*`的映射。

这么做显然能work，但是有（fei）点（chang）蠢。lee给出了两种[非常简洁的递归写法](https://leetcode.com/problems/lowest-common-ancestor-of-deepest-leaves/discuss/334577/JavaC++Python-Two-Recursive-Solution)，其中第一种的思路很简单：写一个递归函数，返回(LCA, 最大深度)，然后对左右子树分别调用这个函数。如果两棵子树的高度不同，则显然最大深度的叶子只存在更深的子树中，那么另一棵子树就不用管了，LCA也不变；否则LCA是当前树根。

### 代码

```cpp
class Solution {
    
    typedef long long int LL;
    
    struct LCA {
        vector<int> G[1002];
        vector<TreeNode*> nodes;
        int maxDepth;
        vector<TreeNode*> deepestLeaves;
        int father[1002][21];
        LL dist[1002];

        // LCA初始化
        LCA(TreeNode* root) {
            nodes = vector<TreeNode*>(1002, NULL);
            maxDepth = -1;
            dfs(root, -1, 0);
            for (int j = 1; j < 21; j++) {
                for (int i = 1; i <= 1000; i++)
                    father[i][j] = father[father[i][j-1]][j-1];
            }
        }

        // 计算父结点和结点深度（用于LCA）
        // 除此之外的部分基本都是LCA模板……
        void dfs(TreeNode* cur, int parent, int depth) {
            if (cur == NULL) return;
            nodes[cur->val] = cur;
            if (cur->left == NULL && cur->right == NULL) {
                if (depth == maxDepth) deepestLeaves.push_back(cur);
                else if (depth > maxDepth) {
                    deepestLeaves.clear();
                    deepestLeaves.push_back(cur);
                    maxDepth = depth;
                }
            }
            father[cur->val][0] = parent == -1 ? cur->val : parent;
            dist[cur->val] = depth;
            dfs(cur->left, cur->val, depth + 1);
            dfs(cur->right, cur->val, depth + 1);
        }

        // 计算x和y的LCA
        int getLCA(int x, int y) {
            if (dist[x] < dist[y]) swap(x, y);
            int d = dist[x] - dist[y];
            for (int i = 20; i >= 0; i--) {
                if (d & (1 << i))
                    x = father[x][i];
            }
            if (x == y) return x;
            for (int i = 20; i >= 0; i--) {
                if (father[x][i] != father[y][i]) {
                    x = father[x][i];
                    y = father[y][i];
                }
            }
            return father[x][0];
        }

        // 根据LCA和深度计算x和y在树中的距离
        LL getDist(int x, int y) {
            int fa = getLCA(x, y);
            return dist[x] + dist[y] - 2 * dist[fa];
        }
    };

    
public:
    TreeNode* lcaDeepestLeaves(TreeNode* root) {
        LCA tool(root);
        int r = tool.deepestLeaves.front()->val;
        for (TreeNode* x: tool.deepestLeaves) {
            r = tool.getLCA(r, x->val);
        }
        return tool.nodes[r];
    }
};
```

## [1124. Longest Well-Performing Interval](https://leetcode.com/problems/longest-well-performing-interval/description/)

标记难度：Medium

提交次数：1/2

代码效率：33.51%（56ms）

### 题意

给定一个只包含1和-1的数组，求最大的和大于0的连续区间的长度。

### 分析

这道题的题目描述中的`[9,9,6]`简直充满了恶意……

我不会做这道题，当初想了很多乱七八糟的方案也没做出来，现在想想，其实一年前似乎做过类似的题。于是在评论区里翻了翻，看到了一道很平凡的题：[560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/description/)。啊，这个贪心我会啊！怎么扩展一下就不会了呢！

仍然参考[lee的解法](https://leetcode.com/problems/longest-well-performing-interval/discuss/334565/JavaC++Python-O(N)-Solution-Life-needs-996-and-669)：遍历数组，计算当前的前缀和，然后进行贪心。显然，有两种情况：第一种是当前的前缀和已经大于0了，这个前缀是合法的；另一种是前缀和<=0了，如果此时存在一个以当前元素结尾的合法区间，在前面一定存在至少一个前缀和是当前的前缀和-1（考虑到每个元素都是1或-1，所以前缀和的变化可以看成是连续的），且选择它是最优的（因为可能的长度最大）。

### 代码

```cpp
class Solution {
public:
    int longestWPI(vector<int>& hours) {
        map<int, int> posMap;
        int cur = 0, ans = 0;
        posMap[0] = -1;
        for (int i = 0; i < hours.size(); i++) {
            if (hours[i] > 8) cur++;
            else cur--;
            // this is crucial
            if (cur > 0) ans = max(ans, i + 1);
            if (posMap.find(cur - 1) != posMap.end())
                ans = max(ans, i - posMap[cur - 1]);
            if (posMap.find(cur) == posMap.end())
                posMap[cur] = i;
        }
        return ans;
    }
};
```

## [1125. Smallest Sufficient Team](https://leetcode.com/problems/smallest-sufficient-team/description/)

标记难度：Hard

提交次数：1/4

代码效率：54.44%（120ms）

### 题意

给定一个要求的编号列表和每个人有的列表，问最少取哪些人，才能覆盖所有要求的列表。

### 分析

这显然是一个状态DP问题，但是我写的时候出了很多麻烦。记`n = people.size()`，`m = req_skills.size()`，我本来设计用`f[n][1 << m]`的状态数组，从前向后转移（而不是从后向前）：

```cpp
f[i+1][j | skill[i+1]] = min(f[i+1][j | skill[i+1]], f[i][j] + 1)
f[i+1][j] = min(f[i+1][j], f[i][j])
```

结果发现要求把具体的选择方案也求出来。这也不是大问题，于是我打算开60 * 65536个`vector<int>`，于是问题来了，我发现内存爆了……当时脑抽了一下，没想到递推时只用到前一步的结果，可以节省空间，于是看了看数据范围（只有60），干脆把`vector<int>`塞到一个`long long int`里去了，于是代码里满地都是`__builtin_popcountll`。

还是lee对这个解法的描述[^lee]说得最好：

[^lee]: [lee215's Solution for Leetcode 1125 - \[Python\] DP Solution](https://leetcode.com/problems/smallest-sufficient-team/discuss/334572/Python-DP-Solution)

>`dp[skill_set]` is a sufficient team to cover the `skill_set`.
>For each people,
>update `his_skill` with all current combinations of `skill_set` in `dp`.

简单来说，存下当前出现过的每种组合的最小队伍，然后每加一个人，尝试把他加到这些队伍里，更新原来的队伍和新出现的组合。

### 代码

```cpp
class Solution {
public:
    vector<int> smallestSufficientTeam(vector<string>& req_skills, vector<vector<string>>& people) {
        map<string, int> skillMap;
        
        for (int i = 0; i < req_skills.size(); i++)
            skillMap[req_skills[i]] = i;
        
        int peopleSkills[65], n = people.size(), m = req_skills.size();
        for (int i = 0; i < n; i++) {
            peopleSkills[i] = 0;
            for (string s: people[i])
                if (skillMap.find(s) != skillMap.end())
                    peopleSkills[i] |= 1 << skillMap[s];
        }
        
        typedef long long int LL;
        LL f[65][65536];
        memset(f, 0, sizeof(f));
        f[0][peopleSkills[0]] |= 1;
        for (int i = 1; i < n; i++) {
            f[i][peopleSkills[i]] = (LL) 1 << i;
            for (int j = 0; j < (1 << m); j++) {
                if (f[i-1][j] == 0) continue;
                
                if (f[i][j] == 0 || __builtin_popcountll(f[i-1][j]) < __builtin_popcountll(f[i][j]))
                    f[i][j] = f[i-1][j];
                int x = j | peopleSkills[i];
                if (f[i][x] == 0 || __builtin_popcountll(f[i-1][j]) + 1 < __builtin_popcountll(f[i][x]))
                    f[i][x] = f[i-1][j] | ((LL) 1 << i);
            }
        }
        
        LL minn = ((LL) 1 << 63) - 1;
        for (int i = 0; i < n; i++)
            if (f[i][(1 << m) - 1] != 0 && __builtin_popcountll(f[i][(1 << m) - 1]) < __builtin_popcountll(minn))
                minn = f[i][(1 << m) - 1];
        
        vector<int> ans;
        for (int i = 0; i < n; i++)
            if (minn & ((LL) 1 << i))
                ans.push_back(i);
        return ans;
    }
};
```
