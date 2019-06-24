---
title: Leetcode Weekly Contest 142总结
urlname: leetcode-weekly-contest-142
toc: true
date: 2019-06-23 20:48:34
updated: 2019-06-24 11:15:00
tags: [Leetcode, Leetcode Contest, alg:Binary Search, alg:Ternary Search]
categories: Leetcode
---

这周的最后一道题简直做到脑壳疼。

<!--more-->

## [1093. Statistics from a Large Sample](https://leetcode.com/problems/statistics-from-a-large-sample/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms

### 题意

用类似哈希表的方式给定一堆数字，求它们的最小值、最大值、平均值、中位数和众数。

### 分析

这下我学到了，“mode”是众数的意思。

……除了中位数之外，其他统计值都没有什么难的。当然，中位数也不难，先计算数字的总数，然后数出位于中间的1/2个数即可。

### 代码

```cpp
class Solution {
    int getKth(int k, vector<int>& count) {
        int sum = 0;
        for (int i = 0; i < count.size(); i++) {
            sum += count[i];
            if (k <= sum) return i;
        }
        return -1;
    }
    
public:
    vector<double> sampleStats(vector<int>& count) {
        int n = count.size(), cnt = 0, modeIdx = 0, sum = 0, minn = -1, maxn = -1;
        for (int i = 0; i < n; i++) {
            cnt += count[i];
            sum += i * count[i];
            if (count[i] > count[modeIdx]) modeIdx = i;
            if (minn == -1 && count[i] > 0) minn = i;
            if (count[i] > 0) maxn = i;
        }
        double median;
        if (cnt % 2 == 0) median = (getKth(cnt / 2, count) + getKth(cnt / 2 + 1, count)) / 2.0;
        else median = getKth(cnt / 2 + 1, count);
        
        return {minn, maxn, (double) sum / cnt, median, modeIdx};
    }
};
```

## [1094. Car Pooling](https://leetcode.com/problems/car-pooling/description/)

标记难度：Medium

提交次数：1/1

代码效率：8ms

### 题意

给定一辆车的容量和若干批游客的数量、上车地点、下车地点，求这辆车能否载动这批游客。

### 分析

因为总站数很少，所以只需计算出每一站净上车的人数，然后模拟，就可以知道是否会出现超载的情况了。

### 代码

```cpp
class Solution {
public:
    bool carPooling(vector<vector<int>>& trips, int capacity) {
        int peopleCnt[1001];
        memset(peopleCnt, 0, sizeof(peopleCnt));
        for (int i = 0; i < trips.size(); i++) {
            peopleCnt[trips[i][1]] += trips[i][0];
            peopleCnt[trips[i][2]] -= trips[i][0];
        }
        int cnt = 0;
        for (int i = 1; i <= 1000; i++) {
            cnt += peopleCnt[i];
            if (cnt > capacity) return false;
        }
        return true;
    }
};
```

## [1095. Find in Mountain Array](https://leetcode.com/problems/find-in-mountain-array/description/)

标记难度：Hard

提交次数：1/1

代码效率：8ms

### 题意

给定一个先单增再单减的数组，要求从中找出`target`出现的位置，如果没有则返回-1。要求查询数组元素的次数小于100次。

### 分析

先用三分查找找到峰的位置，再分别做两次二分查找，一次在峰左边单增的部分，一次在右边单减的部分。想了想，好像还是没有办法直接用三分查找的思路一次把这个数找出来……

二分查找一如既往写得乱七八糟，最后仍然只好用了判断`l`和`l+1`的策略。

### 代码

```cpp
class Solution {
public:
    int findInMountainArray(int target, MountainArray &mountainArr) {
        int n = mountainArr.length();
        int l = 0, r = mountainArr.length() - 1;
        while (l < r) {
            int m1 = l + (r - l) / 3;
            int m2 = l + (r - l) / 3 * 2;
            int x1 = mountainArr.get(m1), x2 = mountainArr.get(m2);
            if (x1 < x2) l = m1 + 1;
            else r = m2 - 1;
        }
        int maxIdx = l, maxn = mountainArr.get(l);
        if (l < n && mountainArr.get(l + 1) > maxn) {
            maxIdx++;
            maxn = mountainArr.get(maxIdx);
        }
        
        if (target == maxn) return maxIdx;
        if (target > maxn) return -1;
        
        l = 0, r = maxIdx;
        while (l < r + 1) {
            int m = (l + r) / 2;
            int x = mountainArr.get(m);
            if (target < x) r = m - 1;
            else if (x < target) l = m + 1;
            else return m;
        }
        l = maxIdx + 1, r = n - 1;
        while (l < r + 1) {
            int m = (l + r) / 2;
            int x = mountainArr.get(m);
            if (target < x) l = m + 1;
            else if (x < target) r = m - 1;
            else return m;
        }
        return -1;
    }
};
```

## [1096. Brace Expansion II](https://leetcode.com/problems/brace-expansion-ii/description/)

标记难度：Hard

提交次数：1/1

代码效率：20ms

### 题意

给定下列文法，求该文法表示的词的集合（注意去重）：

* 单字符是合法的表达式：`"a"`
* 若干个表示连接起来是合法的表达式：`"a{b}c"`
* 若干个表达式用`,`连接，外加`{}`，是合法的表达式：`"{ {a,b},c"}`
* 单字符表达式表示一个仅包含该字符的集合：`R("a") = {"a"}`
* 大括号中用`,`隔开的表达式表示由所有被隔开的表达式组成的集合：`R("{a,b,c}") = {"a","b","c"}`
* 若干个表达式不加`,`连接起来组成的表达式表示这些表达式的笛卡尔积：`R("{a,b}{c,d}") = {"ac","ad","bc","bd"}`

（这里双大括号在Hexo解析里仍然有问题，只好中间加个空格了。）

### 分析

（看到这种题不由得想起编译原理中创造的Decaf语言，然而我已经有点忘了EBNF文法怎么写了）

本来打算用栈写，后来发现写成递归函数好弄得多，所以就递归函数了。

这个文法在代码中解码的方法很多（不过它起码是无二义的……是吧），最后我在代码里写出来的结果是：

* 首先判断是否为纯字母
* 然后判断是否为表达式的乘积
* 最后判断是否为表达式的连接

判断为纯字母而不是判断为**单个**字母的表面原因是节省递归深度，实际原因是按我的写法，没法直接判断纯字母串是否为表达式的乘积。判断字符串是否为表达式的连接比较简单，只要看是否字符串前后都是大括号而且这两个括号成一对就可以了，然后把这对大括号中不在其他大括号内的逗号拆掉；而把表达式乘积字符串拆开比较难，需要分外层的字母和外层的大括号两种情况讨论。

### 代码

```cpp
class Solution {
    
public:
    vector<string> braceExpansionII(string expression) {
        bool isAlpha = true;
        for (char ch: expression)
            if (ch < 'a' || ch > 'z') {
                isAlpha = false;
                break;
            }
        if (isAlpha) return {expression};
        
        // 尝试展开为表达式的乘积
        int paren = 0;
        vector<string> e;
        for (int i = 0; i < expression.size(); i++) {
            if (expression[i] == '{') {
                if (paren == 0 && !(!e.empty() && e.back().size() == 0)) e.push_back(expression.substr(i, 1));
                else e.back() += expression[i];
                paren++;
            }
            else if (expression[i] == '}') {
                paren--;
                e.back() += expression[i];
                if (paren == 0) e.push_back("");
            }
            else {
                if (e.empty()) e.push_back(expression.substr(i, 1));
                else e.back() += expression[i];
            }
        }
        
        if (!e.empty() && e.back().size() == 0) e.pop_back();
        if (e.size() > 1) {
            vector<string> v, v1, v2;
            v = braceExpansionII(e[0]);
            for (int i = 1; i < e.size(); i++) {
                v1 = braceExpansionII(e[i]);
                v2.clear();
                for (string s1: v)
                    for (string s2: v1)
                        v2.push_back(s1 + s2);
                v = v2;
            }
            set<string> sset;
            for (auto s: v) sset.insert(s);
            v.clear();
            for (auto s: sset) v.push_back(s);
            return v;
        }
        
        // 尝试展开为表达式的连接
        e.clear();
        expression = expression.substr(1, expression.length() - 2);
        paren = 0;
        for (int i = 0; i < expression.length(); i++) {
            if (expression[i] == '{') {
                if (e.empty()) e.push_back(expression.substr(i, 1));
                else e.back() += expression[i];
                paren++;
            }
            else if (expression[i] == '}') {
                e.back() += expression[i];
                paren--;
            }
            else if (expression[i] == ',') {
                if (paren == 0) e.push_back("");
                else e.back() += expression[i];
            }
            else {
                if (e.empty()) e.push_back(expression.substr(i, 1));
                else e.back() += expression[i];
            }
        }
        vector<string> v;
        for (string s: e) {
            vector<string> v1(braceExpansionII(s));
            v.insert(v.end(), v1.begin(), v1.end());
        }
        set<string> sset;
        for (auto s: v) sset.insert(s);
        v.clear();
        for (auto s: sset) v.push_back(s);
        return v;
    }
};
```
