---
title: Leetcode Weekly Contest 137总结
urlname: leetcode-weekly-contest-137
toc: true
date: 2019-05-19 20:31:33
updated: 2019-05-20 01:46:00
tags: [Leetcode, Leetcode Contest, alg:Heap, alg:Stack, alg:Dynamic Programming]
categories: Leetcode
---

又是第四题不会做的一天。

<!--more-->

## [1046. Last Stone Weight](https://leetcode.com/problems/last-stone-weight/description/)

标记难度：Easy

提交次数：1/2

代码效率：4ms

### 题意

给定`N`块石头的重量，每次可以取两块重量最大的石头，合成一块新石头（原来两块石头的重量不等，新石头的重量为原来的重量之差）或直接消去。问最后剩下的至多1块石头的重量？

### 分析

水题。直接模拟删除过程即可。用堆维护比较好，暴力也能过。我WA了一次是因为没看到**没有剩余石头时返回0**。

### 代码

```cpp
class Solution {
public:
    int lastStoneWeight(vector<int>& stones) {
        priority_queue<int> pq;
        for (int s: stones)
            pq.push(s);
        while (pq.size() > 1) {
            int x = pq.top(); pq.pop();
            int y = pq.top(); pq.pop();
            if (x == y) continue;
            pq.push(abs(x - y));
        }
        return !pq.empty() ? pq.top() : 0;
    }
};
```

## [1047. Remove All Adjacent Duplicates In String](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string/description/)

标记难度：Easy

提交次数：1/1

代码效率：20ms

### 题意

给定小写英文字母字符串`S`，在`S`上不断移除连续的两个相同字母，直到不能移除为止，问最后剩下的字符串长什么样子？

### 分析

直接把字符串当成一个栈来模拟就行了。显然，在移除两个字母之后，还有可能发生的消去只可能发生在被移除的字母的邻居上。

### 代码

```cpp
class Solution {
public:
    string removeDuplicates(string S) {
        string T;
        for (char c: S) {
            if (T.length() > 0 && T.back() == c) T.pop_back();
            else T.push_back(c);
        }
        return T;
    }
};
```

## [1048. Longest String Chain](https://leetcode.com/problems/longest-string-chain/description/)

标记难度：Medium

提交次数：1/2

代码效率：64ms

### 题意

给定一些由小写英文字母组成的单词，称单词1为单词2的祖先，当且仅当能在单词1中任意位置插入一个字母使得它与单词2相等。称单词链为一个单词序列，其中每一个单词都是后一个单词的祖先。问给定单词能够组成的单词链的最大长度。

### 分析

最开始我WA了一次，因为我没有意识到单词序列中的单词的顺序是任意的。之后我想了想——既然不同单词之间是否为祖先的关系是固定的，那么可以把这道题的问题化为在图上找最长路径。但这样化归我就不会做了……

所以我又想了想，显然，只有长度为`n-1`的单词才会有到长度为`n`的单词的边，所以这是一张层次图。所以只需要从长度为`n-1`的单词向长度为`n`的单词递推就好了……

为了写得更简单一些，甚至还可以不管上面这个关系，直接把所有单词按长度排序，然后按普通的最长递增子序列的方法来做。

### 代码

```cpp
class Solution {
private:
    bool isPredecessor(const string& word1, const string& word2) {
        int n = word1.length();
        if (word2.length() != n + 1) return false;
        int i;
        for (i = 0; i < n && word1[i] == word2[i]; i++);
        for (; i < n; i++) {
            if (word1[i] != word2[i + 1])
                return false;
        }
        return true;
    }
    
    static bool cmp(const string& s1, const string& s2) {
        return s1.length() < s2.length();
    }
    
    int f[1005];
    bool g[1005][1005];

public:
    int longestStrChain(vector<string>& words) {
        int N = words.size();
        sort(words.begin(), words.end(), cmp);
        memset(g, 0, sizeof(g));
        for (int i = 0; i < N; i++)
            for (int j = i + 1; j < N; j++) {
                if (isPredecessor(words[i], words[j])) {
                    g[i][j] = true;
                }
            }
        int ans = 1;
        for (int i = 0; i < N; i++) {
            f[i] = 1;
            for (int j = 0; j < i; j++)
                if (g[j][i])
                    f[i] = max(f[i], f[j] + 1);
            ans = max(f[i], ans);
        }
        return ans;
    }
};
```

## [1049. Last Stone Weight II](https://leetcode.com/problems/last-stone-weight-ii/description/)

标记难度：Medium

提交次数：2/3

代码效率：4ms

### 题意

给定`N`块石头的重量，每次可以**任取两块**石头，合成一块新石头（原来两块石头的重量不等，新石头的重量为原来的重量之差）或直接消去（原来两块石头的重量不等）。问最后剩下的至多1块石头的重量最小是多少？

### 分析

这题我并不会做。于是我去看了看题解，大致看到了两种解法：

* 把题目转化成背包问题：将背包中的物品分成两组，求两组的和之差的最小值[^lee]
* 直接求解前`i`块石头消去后可能得到的重量[^kai]

[^lee]: [lee215's Solution for Leetcode 1049 - \[Java/C++/Python\] Easy Knapsacks DP](https://leetcode.com/problems/last-stone-weight-ii/discuss/294888/JavaC++Python-Easy-Knapsacks-DP)

[^kai]: [Variant of <956 Tallest Billboard>. Python DP with explanation](https://leetcode.com/problems/last-stone-weight-ii/discuss/294906/Variant-of-less956-Tallest-Billboardgreater.-Python-DP-with-explanation)

第一种做法写起来很简单，就是用DP方法求出一个背包的和的所有可能值，然后直接用总和减去这些值，得到对应的另一个背包的和，然后求出对应的剩余价值。显然，每种石头合成方法都可以化归为一种背包消去方法。消去的过程类似下列式子：

```cpp
value = stones[i] + stones[j]
value = value - stones[k] = stones[i] + stones[j] - stones[k]
value = stones[l] - value = - stones[i] - stones[j] + stones[k] + stones[l]
...
value = ± stones[0] ± stones[1] ± stones[2] ± ... ± stones[n-1]
```

显然每一块石头不是被加上去了就是被消去了，把加上去的归到一个背包里，消去的归到另一个背包里即可。

后一种做法看起来更像一般的DP。令`dp[i][diff]`表示`stones[0..i]`是否能通过消去得到一块价值为`diff`的石头。显然，可能有以下三种情况：

```txt
diff' + stones[i] = diff
diff' - stones[i] = diff
stones[i] - diff' = diff
```

因此：

```txt
dp[i][diff] = dp[i-1][diff - stones[i]] || dp[i-1][diff + stones[i]] || dp[i-1][stones[i] - diff]
```

这两种做法的复杂度都是`O(N*S)`，其中`S`是`stones`数组的和。

### 代码

```cpp
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        bool f[3005];
        int sum = 0;
        memset(f, 0, sizeof(f));
        f[0] = 1;
        for (int stone: stones) {
            // 这个顺序是很重要的，不然就把当前的stone加进去了
            for (int i = 3000 - stone; i >= 0; i--)
                if (f[i])
                    f[i + stone] = true;
            sum += stone;
        }
        int ans = 1e9;
        for (int i = 0; i <= sum; i++)
            if (f[i]) {
                ans = min(ans, abs(2 * i - sum));
            }
        return ans;
    }
};
```

```cpp
class Solution {
    int f[30][3005];  // i, diff
    vector<int> stones;
    
    bool dp(int i, int diff) {
        if (i == 0) return diff == stones[0];
        if (diff > 3000 || diff < 0) return false;
        if (f[i][diff] != -1) return f[i][diff];
        f[i][diff] = dp(i - 1, diff + stones[i]) || dp(i - 1, diff - stones[i]) || dp(i - 1, stones[i] - diff);
        return f[i][diff];
    }
    
public:
    int lastStoneWeightII(vector<int>& stones) {
        this->stones = stones;
        memset(f, -1, sizeof(f));
        for (int i = 0; i <= 3000; i++)
            if (dp(stones.size() - 1, i))
                return i;
        return -1;
    }
};
```
