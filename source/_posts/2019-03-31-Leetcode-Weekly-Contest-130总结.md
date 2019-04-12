---
title: Leetcode Weekly Contest 130总结
urlname: Leetcode Weekly Contest 130总结
tags:
  - Leetcode
  - Leetcode Contest
  - 'alg:Monotonic Stack'
  - 'alg:Depth-first Search'
  - 'alg:Math'
categories: Leetcode
toc: true
mathjax: true
date: 2019-03-31 22:16:35
updated: 2019-04-13 02:28:00
---


隔了太久补题，我都不太记得Contest 130做了些啥了，只能随便写写。

<!--more-->

## [1018. Binary Prefix Divisible By 5](https://leetcode.com/problems/binary-prefix-divisible-by-5/description/)

标记难度：Easy

提交次数：1/1

代码效率：16ms

### 题意

给定一个01串`A`，令`A[0..i]`表示一个二进制数，index小表示高位，问这些二进制数中哪些模5余0，哪些模5不余0。

### 分析

直接把这些数依次算出来就行。在途中应当对5取模，因为数组长度最大30000，不取模估计会爆掉。

### 代码

```cpp
class Solution {
public:
    vector<bool> prefixesDivBy5(vector<int>& A) {
        int s = 0;
        vector<bool> ans;
        for (int i = 0; i < A.size(); i++) {
            s = s * 2 + A[i];
            if (s % 5 == 0) ans.push_back(true);
            else ans.push_back(false);
            s %= 5; // 不然int会爆（long long估计也会爆）
        }
        return ans;
    }
};
```

## [1017. Convert to Base -2](https://leetcode.com/problems/convert-to-base-2/description/)

标记难度：Medium

提交次数：1/6

代码效率：4ms

### 题意

将自然数`N`转为“负二”进制。

### 分析

这道题我居然WA了5次，主要是前几次没有搞明白这个(-2)进制到底应该怎么做……

听说这道题在Geeks4Geeks上可以直接搜到[解答](https://www.geeksforgeeks.org/convert-number-negative-base-representation/)，所以在比赛之后受到了一些诟病。说到底，LeetCode好像也没有限定你比赛中能参考哪些内容，所以我觉得也无所谓。如果足够聪明的话，完全可以意识到，这道题的数学形式足够简洁和有意义，以至于可能直接搜到别人的研究结果。

不过不妨假设我从来没听说过这些事情，那么看到这道题之后，我应该问的第一个问题是：“负二”进制是良定义的吗？从题目中，我们可以看出，在“负二”进制中，只有底数是负的，数字还是正的（否则一个“真正”的“负二”进制数应该长这个样子：{% raw %}$(\text{-10-10})_{-2} = (-2)^3 * (-1) + (-2)^1 *(-1) = (10)_{10}${% endraw %}）。这符合[Wikipedia中给出的定义](https://en.wikipedia.org/wiki/Negative_base)。然后存在性和唯一性都可以用和普通进制相同的方法来证明。[^math1][^math2]

[^math1]: [math StackExchange - Prove by induction that every positive integer is represented by some binary number?](https://math.stackexchange.com/questions/1966196/prove-by-induction-that-every-positive-integer-is-represented-by-some-binary-num)

[^math2]: [math StackExchange - Prove that there is only one unique base b representation of any natural number.](https://math.stackexchange.com/questions/2174546/prove-that-there-is-only-one-unique-base-b-representation-of-any-natural-number)

结论：直接按照一般的方法来算就行了。我们通常使用的“一般的方法”可能是长除法（{% raw %}$(15)_{10} = (1111)_{2}${% endraw %}）：

![十进制转二进制](mod2.png)

如果之前没想清楚的话，这里可能会不知道应该把商变成负数还是把余数变成负数（我在这里卡了一会儿）。但很显然，此处我们需要将余数限制在`[0, 1]`范围内，所以商的取值就是固定的了。

![十进制转负二进制](mod-2.png)

其中：

{% raw %}
$$
\begin{aligned}
15 & - & (-2) & * (-7) & = & 1 \\
-7 & - & (-2) & * 4    & = & 1 \\
4  & - & (-2) & * (-2) & = & 0 \\
-2 & - & (-2) & * 1    & = & 0
\end{aligned}
$$
{% endraw %}

在实际求解过程中，需要注意除法取整的方向，核心问题还是将余数限制在`[0, 1]`范围内。

（如果你也像我一样记不清楚C++的那套整数除法理论，就干脆用`double`+`ceil`或`floor`函数吧……）

### 代码

```cpp
class Solution {
public:
    string baseNeg2(int N) {
        if (N == 0) return "0";
        string ans;
        while (N != 0) {
            int x;
            if (N > 0) x = -floor(N / 2.0);  // x为负数，绝对值较小
            else x = ceil(-N / 2.0);  // x为正数，绝对值较大
            int res = N - (-2) * x;
            ans = (res == 1 ? "1" : "0") + ans;
            N = x;
        }
        return ans;
    }
};
```

## [1019. Next Greater Node In Linked List](https://leetcode.com/problems/next-greater-node-in-linked-list/description/)

标记难度：Medium

提交次数：2/8

代码效率：

* 从左到右：240ms
* 从右到左：220ms

### 题意

给定一个链表，为其中的每个元素求出在它右侧比它大的元素中离它最近的元素的值。

### 分析

这道题着实没什么新意，和[503. Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/discuss/98270/)几乎一样（只是数据结构换成了链表）。直接用单调栈来做即可。

以及有一个重要的结论，我应该认真记住：单调栈是对偶的（虽然不知道这么说“对偶”是否合适……），也就是说从两边来做都可以，甚至可以同时得到两个结论。所以这回我又写了两份代码……

### 代码

#### 从左向右的单调栈

这对于这道题来说是比较好的，毕竟提供的是单向链表。

```cpp
class Solution {
public:
    vector<int> nextLargerNodes(ListNode* head) {
        stack<pair<int, int>> stk;  // val, index
        vector<int> ans;
        ListNode* p = head;
        for (int i = 0; p != NULL; i++, p = p->next) {
            ans.push_back(0);
            while (!stk.empty() && stk.top().first < p->val) {
                ans[stk.top().second] = p->val;
                stk.pop();
            }
            stk.emplace(p->val, i);
        }
        return ans;
    }
};
```

#### 从右向左的单调栈

为了方便起见就先化为方便逆序访问的数组了。

```cpp
class Solution {
public:
    vector<int> nextLargerNodes(ListNode* head) {
        vector<int> A;
        stack<pair<int, int>> stk; // val, index
        for (ListNode* p = head; p != NULL; p = p->next)
            A.push_back(p->val);
        int n = A.size();
        vector<int> ans(n, 0);
        for (int i = n - 1; i >= 0; i--) {
            // 注意这个>=（因为题目里要求的是<）
            while (!stk.empty() && A[i] >= stk.top().first)
                stk.pop();
            ans[i] = stk.empty() ? 0 : stk.top().first;
            stk.emplace(A[i], i);
        }
        return ans;
    }
};
```

## [1020. Number of Enclaves](https://leetcode.com/problems/number-of-enclaves/description/)

标记难度：Medium

提交次数：1/1

代码效率：104ms

### 题意

在一个四连通01方格图上计算不与边界连通的1岛的数量。

### 分析

这道题号称是最后一题，实际上水得可以。（画外音：这不就是暴搜吗？）于是我挑战自我（个头），写了次并查集；但是常数好像太大了，完全没有普通的DFS来得快……

不知道是不是并查集自带函数调用常数的原因。

### 代码

```cpp
class Solution {
private:
    pair<int, int> _fa[505][505];
    int n, m;
    
    void init() {
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                _fa[i][j] = make_pair(i, j);
    }
    
    pair<int, int> fa(pair<int, int> p) {
        return p == _fa[p.first][p.second] ? p : _fa[p.first][p.second] = fa(_fa[p.first][p.second]);
    }
    
    void merge(pair<int, int> p1, pair<int, int> p2) {
        p1 = fa(p1);
        p2 = fa(p2);
        _fa[p1.first][p1.second] = p2;
    }
    
public:
    int numEnclaves(vector<vector<int>>& A) {
        n = A.size();
        m = A[0].size();
        init();
        
        // 专门用一个点来表示边界……
        pair<int, int> boundary(501, 501);
        _fa[501][501] = boundary;
        
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (i == 0 || i == n - 1 || j == 0 || j == m - 1)
                    merge(make_pair(i, j), boundary);
                // 和左、上方格连起来即可（右下有那些方格自己连过来）
                if (i > 0 && A[i][j] == 1 && A[i-1][j] == 1)
                    merge(make_pair(i, j), make_pair(i-1, j));
                if (j > 0 && A[i][j] == 1 && A[i][j-1] == 1)
                    merge(make_pair(i, j), make_pair(i, j-1));
            }
        }
        
        int ans = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (A[i][j] == 1 && fa(make_pair(i, j)) != fa(boundary))
                    ans++;
            }
        }
        
        return ans;
    }
};
```
