---
title: Leetcode Weekly Contest 126总结
urlname: leetcode-weekly-contest-126-summary
toc: true
date: 2019-03-03 14:48:05
updated: 2019-03-09 21:02:00
tags: [Leetcode, Leetcode Contest, alg:Stack, alg:Sliding Window, alg:Dynamic Programming]
categories: Leetcode
---

## [1002. Find Common Characters](https://leetcode.com/problems/find-common-characters/description/)

标记难度：Easy

提交次数：1/1

代码效率：36ms

### 题意

给你一堆字符串，找到最大的重复字符的数量。相同的字符可以重复算。

### 分析

最近Leetcode这个题目编号有点乱。

不过这绝对是道水题，统计了每个字符串中每个小写字母的数量之后一起取min即可……

### 代码

```cpp
class Solution {
public:
    vector<string> duplicates(vector<string>& A) {
        map<int, int> charMap[105];
        for (int i = 0; i < A.size(); i++) {
            for (char x: A[i])
                charMap[i][x - 'a']++;
        }
        int minn[26];
        for (int i = 0; i < 26; i++)
            minn[i] = 1e9;
        for (int i = 0; i < A.size(); i++)
            for (int j = 0; j < 26; j++)
                minn[j] = min(minn[j], charMap[i][j]);
        vector<string> ans;
        for (int j = 0; j < 26; j++)
            for (int i = 0; i < minn[j]; i++)
                ans.push_back(string(1, char ('a' + j)));
        return ans;
    }
};
```

## [1003. Check If Word Is Valid After Substitutions](https://leetcode.com/problems/check-if-word-is-valid-after-substitutions/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* 非常暴力的代码：1948ms
* 一般暴力的代码：80ms
* 使用栈的代码：20ms

### 题意

如下定义合法字符串`V`：

* `"abc"`是合法字符串
* 如果`V`是合法字符串且`V = X + Y`（`X`和`Y`均可为空），则`X + "abc" + Y`为合法字符串

判断给定字符串是否是合法字符串。

### 分析

显然可以直接写一个暴力方法：每当找到一个`"abc"`，就把它删掉。（而且可以递归删）

稍微好一点的方法是一次多删几个，因为合法的`"abc"`之间是不会overlap的。

更好的方法是用栈。[^lee]

[^lee]: [lee215's Solution for 1003 - Python Different Solution](https://leetcode.com/problems/check-if-word-is-valid-after-substitutions/discuss/247626/Python-Different-Solution)

### 代码

#### 非常暴力的代码

```cpp
class Solution {
public:
    bool bnfValid(string S) {
        if (S.size() <= 3)
            return S == "abc";
        int n = S.size();
        for (int i = 0; i < n - 3; i++)
            if (S.substr(i, 3) == "abc")
                return bnfValid(S.substr(0, i) + S.substr(i + 3, n - i - 3));
        return false;
    }
};
```

#### 一般暴力的代码

```cpp
class Solution {
public:
    bool isValid(string S) {
        while (true) {
            string a;
            for (char c: S) {
                if (a.length() >= 2 && c == 'c' && a.substr(a.size() - 2, 2) == "ab")
                    a = a.substr(0, a.size() - 2);
                else
                    a += c;
            }
            if (a.length() <= 3) return a.length() == 0;
            if (a == S) return false;
            S = a;
        }
        return true;
    }
};
```

#### 使用栈的代码

其实我觉得这个是正解，上一个代码的本质就是这样的……

```cpp
class Solution {
public:
    bool isValid(string S) {
        vector<char> s;
        for (char c: S) {
            if (c == 'c' && s.size() >= 2 && s[s.size() - 1] == 'b' && s[s.size() - 2] == 'a') {
                s.pop_back();
                s.pop_back();
            }
            else
                s.push_back(c);
        }
        return s.size() == 0;
    }
};
```

## [1004. Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/description/)

标记难度：Medium

提交次数：1/1

代码效率：68ms

### 题意

给定一个01串，可以把其中最多`K`个0改成1，问改完之后最长的全1子串的长度。

### 分析

这显然是个滑动窗口的题目。但问题是怎么写比较好。

我的第一个idea是把连续的01都合并起来，然后去弄滑动窗口。结果我从今天凌晨两点一直调到了三点，这说明我的这个idea本质上可能是非常傻逼的……（当然也可能是凌晨三点不宜写代码。）

今天看了看[lee215的代码](https://leetcode.com/problems/max-consecutive-ones-iii/discuss/247564/JavaC++Python-Sliding-Window)，深感自己之傻逼，为啥人家不到十行就写好了……<del>（因为lee出了第四题啊人家就是吊啊）</del>

不过既然时间不多，我也不再把他的代码抄一遍了，也许：

* 把连续问题化成离散问题不一定是好事
* 滑动窗口维护前沿不变和后沿不变，写法可能会有些差异

### 代码

```cpp
class Solution {
private:
    int calc(vector<pair<int, int>>& b, int start, int end, int sum, int left) {
        int x = 0;
        if (end < b.size() && b[end].first == 0) x += b[end].second;
        if (b[start].first == 0) x += b[start].first;
        else if (start > 0 && b[start - 1].first == 0) x += b[start - 1].second;
        return sum + min(left, x);
    }
    
public:
    int longestOnes(vector<int>& A, int K) {
        // 合并连续01
        vector<pair<int, int>> b;  // 0/1, cnt
        for (int x: A)
            if (b.size() == 0 || b.back().first != x)
                b.emplace_back(x, 1);
            else
                b.back().second++;
        int left = K;
        int end = 0;
        int sum = 0;
        int ans = -1;
        // i是起始位置，end是结束位置，[i, end)
        // sum是当前连续1的数量，left是还没有用来填充0的1的数量
        // 不过sum是不包括零散的1的……（所以才需要calc函数）
        end = -1;
        for (int i = 0; i < b.size(); i++) {
            // 如果i已经超过了当前的结束位置，那么不如另起炉灶，或者干脆不要从i开始的连续的1算了……
            if (i >= end) {
                end = i;
                sum = 0;
                left = K;
                while (end < b.size()) {
                    if (b[end].first == 1) {
                        sum += b[end].second;
                    }
                    else {
                        if (left >= b[end].second) {
                            left -= b[end].second;
                            sum += b[end].second;
                        }
                        else break;
                    }
                    end++;
                }
            }
            // 删除窗口开头元素
            else if (i >= 0 && b[i - 1].first == 1) {
                sum -= b[i - 1].second;
            }
            else if (i >= 0 && b[i - 1].first == 0) {
                sum -= b[i - 1].second;
                left += b[i - 1].second;
            }
            // 更新窗口结尾位置
            while (end < b.size()) {
                if (b[end].first == 1)
                    sum += b[end].second;
                else {
                    if (left < b[end].second) break;
                    left -= b[end].second;
                    sum += b[end].second;
                }
                end++;
            }
            ans = max(calc(b, i, end, sum, left), ans);
        }
        return ans;
    }
};
```

## [1000. Minimum Cost to Merge Stones](https://leetcode.com/problems/minimum-cost-to-merge-stones/description/)

标记难度：Hard

提交次数：1/1

代码效率：32.41%（20ms）

### 题意

给定`N`堆石头，每次可以将其中的连续`K`堆合并为一堆，代价是合并的总石头数。问将这`N`堆石头合并成一堆的最小代价是多少？

### 分析

这题面看起来很像是哈夫曼树了，但是要求只能连续合并，所以显然和哈夫曼没什么关系了。

这道题写得我头昏脑涨，总之最后就是看着[lee的代码](https://leetcode.com/problems/minimum-cost-to-merge-stones/discuss/247567/Python-Top-Down-DP-52ms)怼出来的，所以也没什么可写题解的，就随便写一下我的感想好了……

* 看到这种题就是应该DP，虽然DP方程可能会非常复杂。
* 初始化和转移方程一样，都需要仔细考虑，而且不能瞎考虑。我不知道世界上有没有一种适用于所有DP的思考方法，但估计是没有，所以只能依靠经验了。
* 这道题的转移顺序比较神奇（大概是区间从小到大……），所以写成递归形式比较好。

每当我觉得我已经学会DP了，就会再次看到一堆我无法把握住的DP题，这说明人要活到老学到老。

### 代码

```cpp
class Solution {
private:
    int f[35][35][35];
    int sum[35];
    int n, K;
    
    int getSum(int l, int r) {
        int x = sum[r];
        if (l > 0) x -= sum[l - 1];
        return x;
    }
    
    int calc(int l, int r, int m) {
        if (l > r || m <= 0) return 1e9;
        if (f[l][r][m] != -1) return f[l][r][m];
        if (l == r) {
            if (m == 1) f[l][r][m] = 0;
            else f[l][r][m] = 1e9;
            return f[l][r][m];
        }
        if (m == 1) {
            f[l][r][m] = calc(l, r, K) + getSum(l, r);
            return f[l][r][m];
        }
        f[l][r][m] = 1e9;
        for (int mid = l; mid < r; mid++) {
            f[l][r][m] = min(f[l][r][m], calc(l, mid, 1) + calc(mid + 1, r, m - 1));
        }
        return f[l][r][m];
    }
    
public:
    int mergeStones(vector<int>& stones, int K) {
        n = stones.size();
        for (int i = 0; i < n; i++)
            sum[i] = i == 0 ? stones[i] : sum[i - 1] + stones[i];
        memset(f, -1, sizeof(f));
        this->K = K;
        int ans = calc(0, n - 1, 1);
        return ans >= 1e9 ? -1 : ans;
    }
};
```
