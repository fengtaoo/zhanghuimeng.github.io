---
title: Leetcode Weekly Contest 139总结
urlname: leetcode-weekly-contest-139
toc: true
date: 2019-06-03 03:55:02
updated: 2019-06-04 02:26:00
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Ad-Hoc, alg:Dynamic Programming]
categories: Leetcode
---

这次还是比较简单的。

<!--more-->

## [1071. Greatest Common Divisor of Strings](https://leetcode.com/problems/greatest-common-divisor-of-strings/description/)

标记难度：Easy

提交次数：3/3

代码效率：

* 暴力：26.03%（12ms）
* gcd：95.04%（4ms）

### 题意

求两个字符串的“最大公约字符串”。

### 分析

以这道题的数据规模（字符串长度为1000），最暴力的方法就能过了：对于第二个字符串的每一个子串，判断它能否整除第一个子串。显然我们可以只选择长度为第一个字符串的约束的子串，所以实际的时间复杂度并不大。另一个优化方法是从最长的子串开始，找到最长的子串就返回，这样可以避免重复。

当然，更好的方法是模拟数字的gcd计算字符串的gcd。具体的方法并不完全一样（显然，不太好对字符串求余数），但一种实现方法是，每次只检查长度较短的字符串是否是另一个字符串的前缀，如果是，则继续检查该字符串和另一个字符串去掉前缀之后的结果；否则gcd为空。当较短字符串为空时，较长的字符串就是gcd。[^gcd]

[^gcd]: [Leetcode 1071 Solution - \[Java\] 3 codes: Recursive, iterative and regex w/ brief comments and analysis.](https://leetcode.com/problems/greatest-common-divisor-of-strings/discuss/303781/Java-3-codes:-Recursive-iterative-and-regex-w-brief-comments-and-analysis.)

### 代码

#### 暴力

```cpp
class Solution {
    bool check(string& T, string& x) {
        if (T.length() % x.length() != 0)
            return false;
        int N = T.length(), n = x.length();
        for (int i = 0; i < N; i += n) {
            if (T.substr(i, n) != x)
                return false;
        }
        return true;
    }
    
public:
    string gcdOfStrings(string S, string T) {
        int n = S.length(), m = T.length();
        int maxLen = 0;
        for (int i = 0; i < min(n, m); i++) {
            string x = S.substr(0, i + 1);
            if (check(S, x) && check(T, x))
                maxLen = i + 1;
        }
        return S.substr(0, maxLen);
    }
};
```

### gcd

```cpp
class Solution {
public:
    string gcdOfStrings(string str1, string str2) {
        int m = str1.length(), n = str2.length();
        if (n == 0) return str1;
        if (m < n) return gcdOfStrings(str2, str1);
        if (str1.substr(0, n) != str2) return "";
        return gcdOfStrings(str1.substr(n, m - n), str2);
    }
};
```

## [1072. Flip Columns For Maximum Number of Equal Rows](https://leetcode.com/problems/flip-columns-for-maximum-number-of-equal-rows/description/)

标记难度：Medium

提交次数：1/1

代码效率：64.37%（216ms）

### 题意

给定一个只包含0和1的矩阵，可以选择矩阵的若干列，把它们的值翻转，问翻转后最多有几行会变成全0或全1？

### 分析

显然，每一行变成全0或全1只有两种可能：0全都被翻转成1，或者1全都被翻转成0。所以只需要用一个哈希表统计每种翻转方法一共对应了多少行。虽然每一行有两种可能的翻转方法，但很显然，这两种方法不会被重复计算。

另一种更有趣的方法是，把每一行都翻转到第一个位置的数为0，然后再进行统计，这样就可以减少总共需要的统计数量了。[^lee]

[^lee]: [lee215's solution for Leetcode 1072 - \[Python\] 1 Line](https://leetcode.com/problems/flip-columns-for-maximum-number-of-equal-rows/discuss/303752/Python-1-Line)

### 代码

```cpp
class Solution {
public:
    int maxEqualRowsAfterFlips(vector<vector<int>>& A) {
        map<vector<int>, int> mmap;
        int n = A.size(), m = A[0].size();
        for (int i = 0; i < n; i++) {
            mmap[A[i]]++;
            vector<int> B(A[i]);
            for (int j = 0; j < m; j++)
                B[j] = 1 - B[j];
            mmap[B]++;
        }
        int ans = 0;
        for (auto& p: mmap)
            ans = max(ans, p.second);
        return ans;
    }
};
```

## [1073. Adding Two Negabinary Numbers](https://leetcode.com/problems/adding-two-negabinary-numbers/description/)

标记难度：Medium

提交次数：1/3

代码效率：77.78%（12ms）

### 题意

将两个-2进制数相加。加数的长度不超过1000。

### 分析

开始做的时候没看到1000，写了一个转成十进制相加再转回来的方法，十分愚蠢。

前段时间的比赛中（[Leetcode Weekly Contest 130总结](https://zhanghuimeng.github.io/post/leetcode-weekly-contest-130/)）也用到了-2进制。这种题其实也不太难，搞清楚如何适当地修改加法规则就可以了（那倒是和第一题有点像）。

假定当前的-2进制数`a`可以用`a[0] * (-2)^0 + a[1] * (-2)^1 + ... + a[n-1] * (-2)^(n-1)`来表示。在加法过程中，如果`a[i] < 0`，则需要从`a[i + 1]`借位。此时：

```cpp
a[0] * (-2)^0 + ... + a[i]     * (-2)^i + a[i+1]     * (-2)^(i+1) + ... + a[n-1] * (-2)^(n-1) =
a[0] * (-2)^0 + ... + (a[i]+2) * (-2)^i + (a[i+1]+1) * (-2)^(i+1) + ... + a[n-1] * (-2)^(n-1)
```

可以看出来，“借位”实际上是+1。同理，如果`a[i] > 1`，则需要向`a[i + 1]`进位：

```cpp
a[0] * (-2)^0 + ... + a[i]     * (-2)^i + a[i+1]     * (-2)^(i+1) + ... + a[n-1] * (-2)^(n-1) =
a[0] * (-2)^0 + ... + (a[i]-2) * (-2)^i + (a[i+1]-1) * (-2)^(i+1) + ... + a[n-1] * (-2)^(n-1)
```

“进位”实际上是-1。

然后直接参照普通的高精度加法写就可以了。

### 代码

```cpp
class Solution {
public:
    vector<int> addNegabinary(vector<int>& A, vector<int>& B) {
        int n = A.size(), m = B.size();
        reverse(A.begin(), A.end());
        reverse(B.begin(), B.end());
        vector<int> C;
        C.push_back(0);
        for (int i = 0; ; i++) {
            C[i] += (i < n ? A[i] : 0) + (i < m ? B[i] : 0);
            if (i >= n && i >= m && C[i] == 0)
                break;
            C.push_back(0);
            if (C[i] > 1) {
                C[i + 1]--;
                C[i] -= 2;
            }
            else if (C[i] < 0) {
                C[i + 1]++;
                C[i] += 2;
            }
        }
        while (C.size() > 1 && C.back() == 0) C.pop_back();
        reverse(C.begin(), C.end());
        return C;
    }
};
```

## [1074. Number of Submatrices That Sum to Target](https://leetcode.com/problems/number-of-submatrices-that-sum-to-target/description/)

标记难度：Hard

提交次数：1/4

代码效率：73.10%（4084ms）

### 题意

给定一个矩阵，求其中和等于`target`的子矩阵的数量。

### 分析

这道题的思路和一维情形差不多。枚举子矩阵两列的边界，然后在行的意义上就降维到一维情形了。

至于一维情形，就是遍历数组，用一个哈希表记录已经访问过的前缀和，然后每次在哈希表中查找当前前缀和减`target`。

为了加速计算，可以计算出每行的前缀和。

### 代码

```cpp
class Solution {
public:
    int numSubmatrixSumTarget(vector<vector<int>>& A, int target) {
        typedef long long int LL;
        int n = A.size(), m = A[0].size();
        int ans = 0;
        
        int linePrefixSum[300][300];  // 每行的前缀和
        for (int i = 0; i < n; i++) {
            linePrefixSum[i][0] = A[i][0];
            for (int j = 1; j < m; j++)
                linePrefixSum[i][j] = linePrefixSum[i][j-1] + A[i][j];
        }
        
        for (int i = 0; i < m; i++) {
            for (int j = i; j < m; j++) {
                LL sum = 0;
                unordered_map<LL, int> lineMap;
                lineMap[0] = 1;
                for (int k = 0; k < n; k++) {
                    sum += linePrefixSum[k][j] - (i > 0 ? linePrefixSum[k][i-1] : 0);
                    auto p = lineMap.find(sum - target);
                    if (p != lineMap.end())
                        ans += p->second;
                    lineMap[sum]++;
                }
            }
        }
        return ans;
    }
};
```
