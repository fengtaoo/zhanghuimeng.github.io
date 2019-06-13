---
title: Leetcode Weekly Contest 140总结
urlname: leetcode-weekly-contest-140
toc: true
mathjax: true
date: 2019-06-11 18:50:28
updated: 2019-06-13 18:06:00
tags: [Leetcode, Leetcode Contest, alg:Tree, alg:String, alg:Math, alg:Stack, alg:Greedy]
categories: Leetcode
---

这周的比赛内容可以说是有点诡异。

<!--more-->

## [1078. Occurrences After Bigram](https://leetcode.com/problems/occurrences-after-bigram/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms

### 题意

给定一个用空格分隔的字符串和两个词，找出字符串中所有连续出现这两个词时后面跟着的第三个词。

### 分析

虽然题目描述听起来有些怪异，但搞明白之后，原理还是非常简单的；唯一的问题可能还是，C++不自带字符串split功能……

### 代码

```cpp
class Solution {
    vector<string> split(const string& str, const string& delim)
    {
        vector<string> tokens;
        size_t prev = 0, pos = 0;
        do
        {
            pos = str.find(delim, prev);
            if (pos == string::npos) pos = str.length();
            string token = str.substr(prev, pos-prev);
            if (!token.empty()) tokens.push_back(token);
            prev = pos + delim.length();
        }
        while (pos < str.length() && prev < str.length());
        return tokens;
    }
    
public:
    vector<string> findOcurrences(string text, string first, string second) {
        vector<string> ans;
        vector<string> words = split(text, " ");
        for (int i = 0; i < words.size() - 1; i++) {
            if (i > 0 && words[i-1] == first && words[i] == second)
                ans.push_back(words[i+1]);
        }
        return ans;
    }
};
```

## [1079. Letter Tile Possibilities](https://leetcode.com/problems/letter-tile-possibilities/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* DFS：44.36%（64ms）
* 数学：0ms

### 题意

给定7个大写字母，问用这些字母最多能组成多少种不同的非空序列。

### 分析

在数据大小只有7的前提下，我猜怎么做都可以，于是我就直接DFS枚举去重了，时间也没有很慢。

当然，稍微好一点的思路是先考虑选择每种字母各多少个，选好之后，再考虑将这些字母排列之后能够产生多少种序列。假定`cnt[i]`表示第`i`个字母的数量，则选择字母的总方案数为`(cnt[0] + 1) * (cnt[1] + 1) * ... * (cnt[25] + 1)`（+1是因为可以不选）；选好之后，排列总数为

$$
\frac{(\sum_{i=0}^{25} cnt'[i])!}{\prod_{i=0}^{25} cnt'[i]!}
$$

（显然，这是基础的组合数学知识）

一种我比较喜欢的解法[^prod]在此处采用了一种很聪明的写法：直接用类似于进制转换的方法枚举出每种字母选了多少个（而不是用DFS再穷举）。

[^prod]: [Simple Python solution with thinking process (works for much longer input)](https://leetcode.com/problems/letter-tile-possibilities/discuss/308216/Simple-Python-solution-with-thinking-process-(works-for-much-longer-input))

### 代码

#### DFS

```cpp
class Solution {
    
    unordered_set<string> seqs;
    
    void dfs(vector<int> cnt, string x) {
        seqs.insert(x);
        for (int i = 0; i < 26; i++) {
            if (cnt[i] > 0) {
                cnt[i]--;
                dfs(cnt, x + (char) ('A' + i));
                cnt[i]++;
            }
        }
    }
    
public:
    int numTilePossibilities(string tiles) {
        vector<int> cnt(26, 0);
        for (char c: tiles)
            cnt[c - 'A']++;
        dfs(cnt, "");
        return seqs.size() - 1;
    }
};
```

#### 数学

```cpp
class Solution {
    int fac(int x) {
        int p = 1;
        while (x > 1)
            p *= x--;
        return p;
    }
    
public:
    int numTilePossibilities(string tiles) {
        int cnt[26];
        memset(cnt, 0, sizeof(cnt));
        for (char ch: tiles)
            cnt[ch - 'A']++;
        int f = 1;
        for (int i = 0; i < 26; i++)
            f *= cnt[i] + 1;
        int freq[26];
        int ans = 0;
        for (int i = 1; i < f; i++) {
            int x = i, sum = 0;
            for (int j = 0; j < 26; j++) {
                freq[j] = x % (cnt[j] + 1);
                sum += freq[j];
                x /= cnt[j] + 1;
            }
            sum = fac(sum);
            for (int j = 0; j < 26; j++)
                sum /= fac(freq[j]);
            ans += sum;
        }
        return ans;
    }
};
```

## [1080. Insufficient Nodes in Root to Leaf Paths](https://leetcode.com/problems/insufficient-nodes-in-root-to-leaf-paths/description/)

标记难度：Medium

提交次数：1/5

代码效率：81.63%（44ms）

### 题意

给定一棵二叉树，同时从树上删去所有满足以下条件的结点：所有经过该结点的从根到叶子的路径上结点值的总和均小于`limit`。

### 分析

这道题的描述可真是太绕了。首先可以发现一个事实：因为所有经过孩子的路径必然会经过父亲，所以如果父亲不合法，孩子必然也不合法，最后并不会删出来孤儿结点之类的（虽然这个事实没什么用）。下一个问题就是怎么合理地把这些描述塞到一次对树的遍历中了。

* 因为需要求路径上结点的和，所以需要加一个`sum`参数
* 因为需要删除结点，所以需要返回一个指针，不删除时将当前结点返回，如果删除则返回`NULL`
* 因为只有在底层遍历结束时才能够知道路径的和，所以需要在发现是叶子结点时返回路径的和
* ……

因为C++并不能返回Tuple，所以最后只好把一些内容塞到了传引用的参数里。

最后需要注意的一点是（因为我用的是C++），删除结点之后不要自己把它delete掉，否则会RE，大概Leetcode内部自己delete过了……

### 代码

```cpp
class Solution {
    
    // sum：到root为止的sum
    // faMaxSum：经过父结点的路径的最大sum
    // myMaxSum：经过当前结点的路径的最大sum
    TreeNode* dfs(TreeNode* root, int limit, int sum, int& faMaxSum) {
        if (root == NULL) return NULL;
        sum += root->val;
        int myMaxSum = -2e9;
        // 在删除结点之前判断是否为叶子
        if (root->left == NULL && root->right == NULL)
            myMaxSum = max(myMaxSum, sum);
        root->left = dfs(root->left, limit, sum, myMaxSum);
        root->right = dfs(root->right, limit, sum, myMaxSum);
    
        faMaxSum = max(faMaxSum, myMaxSum);

        if (myMaxSum < limit)
            return NULL;
        return root;
    }
    
public:
    TreeNode* sufficientSubset(TreeNode* root, int limit) {
        int maxSum = -2e9;
        return dfs(root, limit, 0, maxSum);
    }
};
```

## [1081. Smallest Subsequence of Distinct Characters](https://leetcode.com/problems/smallest-subsequence-of-distinct-characters/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* O(N^2)：22.33%（8ms）
* 栈：0ms

### 题意

给定一个字符串，要求从中选择满足以下条件且字典序最小的子序列：包含原串中每个字母一次，且仅包含一次。

### 分析

我开始的时候随便脑补了一种做法：

* 从右到左遍历字符串，记录满足以下条件中对应的字母最小且最靠左的index：从index开始的后缀中已经包含了原串中的所有字母
* 记录index对应的字母，把index左侧的字符串丢掉，在index右侧的字符串中也去除该字母
* 回到第一步，直到字符串为空为止

这是一种贪心的做法。在这种做法中，每次选择的都是“下一个字母”，所以为了使字典序最小，需要选择“能够选的字母中最小的”。显然，只有对应后缀中仍然包含所有剩余字母的字母可以选；在较小的字母中仍然选择靠左的是因为这样右侧剩下的字母会多一些。

当然，这么做不够好。更好的做法是用一个栈来存储结果：如果下一个字母比当前栈底要小，且之后栈底字母还会再次出现，则可以把栈底弹出。这样做更像是在尝试寻找一个单调递增的序列，虽然很显然不一定能找到。[^lee]

[^lee]: [lee215's Solution for Leetcode 1081 - \[Java/Python\] Stack Solution O(N)](https://leetcode.com/problems/smallest-subsequence-of-distinct-characters/discuss/308210/JavaPython-Stack-Solution-O%28N%29)

### 代码

#### O(N^2)的贪心

```cpp
class Solution {
    
public:
    string smallestSubsequence(string text) {
        string ans;
        while (!text.empty()) {
            unordered_set<char> chSet;
            int maxCnt = 0, minPos = -1;
            char minChar = 127;
            for (int i = text.size() - 1; i >= 0; i--) {
                chSet.insert(text[i]);
                if (chSet.size() > maxCnt || text[i] <= minChar) {
                    maxCnt = chSet.size();
                    minChar = text[i];
                    minPos = i;
                }
            }
            // 丢掉左边的部分
            text = text.substr(minPos + 1, text.size() - minPos - 1);
            ans += minChar;
            string tmp;
            // 在右边去掉所有minChar
            for (char ch: text)
                if (ch != minChar)
                    tmp += ch;
            text = tmp;
        }
        return ans;
    }
};
```

#### 栈

```cpp
class Solution {
public:
    string smallestSubsequence(string text) {
        int cnt[26];
        memset(cnt, 0, sizeof(cnt));
        for (char c: text)
            cnt[c - 'a']++;
        bool visited[26];
        memset(visited, 0, sizeof(visited));
        string ans;
        for (char c: text) {
            cnt[c - 'a']--;
            if (visited[c - 'a']) continue; // 为了保证结果，这是必要的
            while (!ans.empty() && ans.back() > c && cnt[ans.back() - 'a'] > 0) {
                visited[ans.back() - 'a'] = false;
                ans.pop_back();
            }
            ans.push_back(c);
            visited[c - 'a'] = true;
        }
        return ans;
    }
};
```
