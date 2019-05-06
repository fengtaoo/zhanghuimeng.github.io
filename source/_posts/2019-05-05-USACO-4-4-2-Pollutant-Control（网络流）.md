---
title: 'USACO 4.4.2: Pollutant Control（网络流）'
urlname: usaco-4-4-2-pollutant-control
toc: true
mathjax: true
date: 2019-05-05 22:41:23
updated: 2019-05-06 15:21:00
tags: [USACO, alg:Network Flow]
categories: USACO
---

## 题意

见[洛谷P1344 追查坏牛奶 Pollutant Control](https://www.luogu.org/problemnew/show/P1344)。

给定网络流图、源点和汇点，求最大流的值，以及边数最小且边的编号字典序最小的最小割。

<!--more-->

## 分析

单纯算个最大流很简单（特别是在`N=32`的情况下）。于是我去学习了一下Dinic算法，写（chao）了一顿。

算个最小割也很简单。只要在算完最大流之后的残量网络上，从源点出发BFS/DFS，然后很快就可以把整个图分成S和T两个集合；从S出发，到T结束的所有饱和边就组成了最小割。（这么想来，这道题有点像[USACO 4.3.2: Street Race（图）](/post/usaco-4-3-2-street-race/)）

但是算个边数最少的最小割比较难。众所周知，最小割和最大流并不是唯一的。解决方法是修改图中的容量：记原来的容量为函数$c(\cdot)$，现在的容量为$c'(\cdot) = |E + 1|c(\cdot) + 1$，则修改容量之后的最小割$Mincut(c')$仍然是原图的最小割，且边数最小。原因很容易理解：

$$Mincut(c') = \sum_{e \in P} c'(e) = \sum_{e \in P} (|E + 1|c(e) + 1) = |E + 1|\sum_{e \in P} c(e) + |P|$$

显然有$|P| \leq |E + 1|$，因此$Mincut(c')$最小时，$\sum_{e \in P} c(e)$一项必定也是最小的，且$|P|$也最小。[^pine]

[^pine]: [pinkpurplepineapples - Min Cut with fewest edges](https://pinkpurplepineapples.wordpress.com/2016/11/08/min-cut-with-minimal-edge-count/)

算个边数最少且字典序最小的最小割好像就更难了。[^int]在下图这个测试样例中，这看起来是个很大的问题：

![测试样例](example.png)

[^int]: 此处原文为“If there are still multiple sets, choose the one whose initial routes have the smallest index.”。说实话，我不是很确定这句话到底是不是字典序的意思，但大概是吧。

所以我们需要继续修改图中的容量[^prob]，变成$c'(\cdot) = |E + 1|2^{|E|}c(\cdot) + 2^{|E|} + 2^i$，其中$i$是边的编号。此时有

[^prob]: 一种能通过所有样例的修改方法是$c'(\cdot) = |E + 1||K|c(\cdot) + |K| + i$，但我觉得只加一个$i$无法体现出字典序，因为可能有1+6 > 2+3这种情况。我并没有实际去构造这样的反例，当然也没有证明这样的例子一定是不存在的。USACO上的题解说：“Then I split the first route in the min cut into two with the same source and end to check whether it can be replaced by routes with a smaller initial index. However, this solution won't work for multi-substitute cases, like the one below.”意思是他基本上也没管这个问题……

{% raw %}
$$
\begin{aligned}
Mincut(c') &= \sum_{e \in P} c'(e) \\
&= \sum_{e \in P} (|E + 1|2^{|E|}c(\cdot) + 2^{|E|} + 2^i) \\
&= |E + 1|2^{|E|}\sum_{e \in P} c(e) + 2^{|E|}|P| + \sum_{e \in P}2^i
\end{aligned}
$$
{% endraw %}

显然这三项是互不影响的：$|P| < |E + 1|$，且$\sum 2^i < 2^{|E|}$。因此最小化上式就相当于在计算最小割的前提下（$|E + 1|2^{|E|}\sum_{e \in P} c(e)$）最小化最小割的边数（$2^{|E|}|P|$），在最小化边数的前提下再最小化字典序（$\sum_{e \in P}2^i$）。

然后就出现了一个小问题：题目范围是$E \leq 1000$，$|c(e)| \leq 2000000$，因此现在最大的权值就变成了差不多$2^{1000} \cdot 1000 \cdot 1000$这么大，必须要用高精度了。于是我又遇到了一点小问题：位数开多了USACO的内存会炸（我没压位），而且需要注意，算`pow`的时候不要爆数组长度……

我不确定这么做是不是最优解，但我觉得这个解法是正确的，所以姑且就这样吧。

## 代码

前面三百多行都是高精度，中间是网络流……

```cpp
/*
ID: zhanghu15
TASK: milk6
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>
#include <map>
#include <queue>

using namespace std;
typedef long long int LL;
typedef unsigned long long int ULL;

ofstream fout("milk6.out");
ifstream fin("milk6.in");

struct BigInt {
    static const int maxn = 355;
    int digits[maxn];
    int len;
    int sign;

    void setByInt(LL x) {
        memset(digits, 0, sizeof(digits));
        len = 0;
        if (x < 0) sign = -1, x = -x;
        else sign = 1;
        while (x > 0) {
            digits[len++] = x % 10;
            x /= 10;
        }
    }

    void setByChar(const char* c) {
        // TODO：没有考虑符号
        memset(digits, 0, sizeof(digits));
        sign = 1;
        len = strlen(c);
        for (int i = 0; i < len; i++)
            digits[len - i - 1] = c[i] - '0';
    }

    void setByBigInt(const BigInt& a) {
        this->len = a.len;
        this->sign = a.sign;
        memcpy(this->digits, a.digits, sizeof(a.digits));
    }

    LL toLL() const {
        LL sum = 0;
        for (int i = len - 1; i >= 0; i--)
            sum = sum * 10 + digits[i];
        return sum;
    }

    // abs(a) < abs(b)
    friend bool lessThanSameSign(const BigInt& a, const BigInt& b) {
        if (a.len < b.len) return true;
        if (a.len > b.len) return false;
        for (int i = a.len - 1; i >= 0; i--) {
            if (a.digits[i] < b.digits[i]) return true;
            else if (a.digits[i] > b.digits[i]) return false;
        }
        return false;
    }

    // abs(a) > abs(b)
    friend bool moreThanSameSign(const BigInt& a, const BigInt& b) {
        if (a.len > b.len) return true;
        if (a.len < b.len) return false;
        for (int i = a.len - 1; i >= 0; i--) {
            if (a.digits[i] > b.digits[i]) return true;
            else if (a.digits[i] < b.digits[i]) return false;
        }
        return false;
    }

    // abs(a) + abs(b)
    friend BigInt plusSameSign(const BigInt& a, const BigInt& b) {
        BigInt c;
        c.sign = 1;
        c.len = max(a.len, b.len);
        for (int i = 0; i < c.len; i++) {
            c.digits[i] += a.digits[i] + b.digits[i];
            c.digits[i + 1] += c.digits[i] / 10;
            c.digits[i] %= 10;
        }
        while (c.digits[c.len] != 0)
            c.len++;
        return c;
    }

    // abs(a) - abs(b)
    // guaranteed abs(a) >= abs(b)
    friend BigInt minusSameSign(const BigInt& a, const BigInt& b) {
        BigInt c;
        c.sign = 1;
        c.len = max(a.len, b.len);
        for (int i = 0; i < c.len; i++) {
            c.digits[i] += a.digits[i] - b.digits[i];
            if (c.digits[i] < 0) {
                c.digits[i + 1]--;
                c.digits[i] += 10;
            }
        }
        while (c.len > 0 && c.digits[c.len - 1] == 0) c.len--;
        return c;
    }

    friend void divisionByInt(const BigInt& a, LL b, BigInt& c, LL& temp) {
        // TODO: 负数除法
        if (lessThanSameSign(a, b)) {
            c = 0;
            temp = a.toLL();
            return;
        }
        int sign = 1;
        if (b < 0) sign = -1, b = -b;
        c.len = a.len;
        temp = 0;
        while (temp < b && c.len >= 0) {
            temp = temp * 10 + a[c.len - 1];
            c.len--;
        }
        c.len++;
        for (int i = c.len - 1; i >= 0; i--) {
            c[i] = temp / b;
            temp %= b;
            if (i > 0) temp = temp * 10 + a[i - 1];
        }
        c.sign = a.sign * sign;
    }

    /*------------constructors------------*/

    BigInt() {
        // 0的len为0……
        len = 0;
        sign = 1;
        memset(digits, 0, sizeof(digits));
    }

    BigInt(LL x) {
        this->setByInt(x);
    }

    explicit BigInt(const char* c) {
        this->setByChar(c);
    }

    BigInt(const BigInt& a) {
        this->setByBigInt(a);
    }

    /*------------basic operators------------*/

    friend istream& operator >> (istream& in, BigInt& a) {
        char c[maxn];
        in >> c;
        a.setByChar(c);
        return in;
    }

    friend ostream& operator << (ostream& out, const BigInt& a) {
        if (a.sign == -1) out << '-';
        for (int i = a.len - 1; i >= 0; i--)
            out << a.digits[i];
        if (a.len == 0) out << 0;
        return out;
    }

    friend bool operator < (const BigInt& a, const BigInt& b) {
        if (a.sign == b.sign) {
            if (a.sign == 1) return lessThanSameSign(a, b);
            else return !lessThanSameSign(a, b);
        }
        return a.sign < b.sign;
    }

    friend bool operator > (const BigInt& a, const BigInt& b) {
         if (a.sign == b.sign) {
            if (a.sign == 1) return moreThanSameSign(a, b);
            else return !moreThanSameSign(a, b);
        }
        return a.sign > b.sign;
    }

    friend bool operator == (const BigInt& a, const BigInt& b) {
        if (a.sign != b.sign || a.len != b.len) return false;
        for (int i = 0; i < a.len; i++)
            if (a.digits[i] != b.digits[i])
                return false;
        return true;
    }

    friend bool operator == (const BigInt& a, const LL& b) {
        BigInt nb(b);
        return a == nb;
    }

    friend bool operator != (const BigInt& a, const LL& b) {
        return !(a == b);
    }

    // 单目-运算符
    BigInt operator - () const {
        BigInt a(*this);
        if (a != 0)
            a.sign = -a.sign;
        return a;
    }

    // implements the main logic of +
    friend BigInt operator + (const BigInt& a, const BigInt& b) {
        if (a.sign == b.sign) {
            BigInt c(plusSameSign(a, b));
            c.sign = a.sign;
            return c;
        }
        BigInt c;
        if (lessThanSameSign(a, b)) {
            c = minusSameSign(b, a);
            if (c == 0) return c;
            if (a.sign == 1 && b.sign == -1)
                c.sign = -1;
            else if (a.sign == -1 && b.sign == 1)
                c.sign = 1;
        }
        else {
            c = minusSameSign(a, b);
            if (c == 0) return c;
            if (a.sign == 1 && b.sign == -1)
                c.sign = 1;
            else if (a.sign == -1 && b.sign == 1)
                c.sign = -1;
        }
        return c;
    }

    friend BigInt operator - (const BigInt& a, const BigInt& b) {
        BigInt minusb = -b;
        return a + minusb;
    }

    friend BigInt operator * (const BigInt& a, const BigInt& b) {
        if (a == 0 || b == 0) return 0;
        BigInt c;
        c.sign = a.sign * b.sign;
        c.len = a.len + b.len - 1;
        for (int i = 0; i < a.len; i++) {
            for (int j = 0; j < b.len; j++) {
                c[i + j] += a[i] * b[j];
                c[i + j + 1] += c[i + j] / 10;
                c[i + j] %= 10;
            }
        }
        while (c[c.len] > 0)
            c.len++;
        return c;
    }

    friend BigInt operator / (const BigInt& a, LL b) {
        // TODO: b <= 0?
        // TODO: 负数舍入？
        BigInt c;
        LL residual;
        divisionByInt(a, b, c, residual);
        return c;
    }

    friend LL operator % (const BigInt& a, LL b) {
        BigInt c;
        LL residual;
        divisionByInt(a, b, c, residual);
        return residual;
    }

    /*------------syntax sugars------------*/

    int& operator[](int index)
    {
        return digits[index]; 
    }

    const int& operator[](int index) const
    {
        return digits[index]; 
    }

    friend void operator += (BigInt& a, const BigInt& b) {
        a = a + b;
    }

    friend void operator -= (BigInt& a, const BigInt& b) {
        a = a - b;
    }

    friend void operator *= (BigInt& a, const BigInt& b) {
        a = a * b;
    }

    friend void operator /= (BigInt& a, const LL& b) {
        a = a / b;
    }

    friend void operator %= (BigInt& a, const LL& b) {
        a = a % b;
    }

    /*------------math functions------------*/

    friend BigInt min(const BigInt& a, const BigInt& b) {
        if (a < b) return a;
        return b;
    }

    friend BigInt abs(const BigInt& a) {
        BigInt c(a);
        c.sign = 1;
        return c;
    }

    friend BigInt pow(const BigInt& x, BigInt a) {
        BigInt ans = 1;
        BigInt p = x;
        LL res;
        BigInt b;
        while (a > 0) {
            b.setByInt(0);
            divisionByInt(a, 2, b, res);
            if (res == 1) ans *= p;
            if (b == 0) break;
            p = p * p;
            a = b;
        }
        return ans;
    }
};

struct Edge {
    int from, to;
    BigInt cap, flow;
    Edge(int f, int t, BigInt c, BigInt w) {
        from = f;
        to = t;
        cap = c;
        flow = w;
    }
};

struct Dinic {
    int n, m, s, t;
    vector<Edge> edges;
    vector<int> G[5005];
    bool visited[5005];
    int dist[5005];
    int cur[5005];

    void addEdge(int from, int to, BigInt cap) {
        edges.emplace_back(from, to, cap, 0);
        edges.emplace_back(to, from, 0, 0);
        m = edges.size();
        G[from].push_back(m - 2);
        G[to].push_back(m - 1);
    }

    bool bfs() {
        memset(visited, 0, sizeof(visited));
        queue<int> q;
        q.push(s);
        visited[s] = true;
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            for (int x: G[u]) {
                Edge& e = edges[x];
                // residual network
                if (!visited[e.to] && e.cap > e.flow)  {
                    visited[e.to] = true;
                    dist[e.to] = dist[u] + 1;
                    q.push(e.to);
                }
            }
        }
        return visited[t];
    }

    // x: 当前结点
    // a: 当前路径上所有弧的最小残量
    BigInt dfs(int x, BigInt a) {
        // cout << "dfs: " << x << ' ' << a << endl;
        if (x == t || a == 0) return a;
        BigInt flow = 0, f;
        // 保存每个结点x正在考虑的弧cur[x]，防止重复计算
        for (int& i = cur[x]; i < G[x].size(); i++) {
            Edge& e = edges[G[x][i]];
            // cout << e.cap << ' ' << e.flow << endl;
            if (dist[x] + 1 == dist[e.to] && (f = dfs(e.to, min(a, e.cap - e.flow))) > 0) {
                e.flow += f;
                edges[G[x][i] ^ 1].flow -= f;
                flow += f;
                a -= f;
                if (a == 0) break;
            }
        }
        return flow;
    }

    BigInt maxflow() {
        BigInt flow = 0;
        BigInt INF = pow((BigInt) 10, 350);
        while (bfs()) {
            for (int i = 1; i <= n; i++) {
                // cout << i << ' ' << dist[i] << endl;
            }
            for (auto&e: edges) {
                // cout << e.from << ' ' << e.to <<' ' << e.cap << ' ' << e.flow << endl;
            }
            memset(cur, 0, sizeof(cur));
            flow += dfs(s, INF);
            // cout << flow << endl;
        }
        return flow;
    }

    // 默认编号为1-n
    vector<int> mincut() {
        bfs();
        vector<int> cut;
        for (int i = 0; i < edges.size(); i++) {
            Edge& e = edges[i];
            if (e.cap > 0 && e.cap == e.flow && visited[e.from] && !visited[e.to])
                cut.push_back(i / 2);
        }
        return cut;
    }
};

Dinic dinic;
LL N, M;

int main() {
    fin >> N >> M;
    dinic.n = N;
    dinic.s = 1;
    dinic.t = N;
    BigInt K = pow((BigInt) 2, M), p = 1;
    LL s, e, c;
    vector<LL> origEdges;
    for (int i = 0; i < M; i++) {
        fin >> s >> e >> c;
        dinic.addEdge(s, e, M * K * c + K + p);
        p *= 2;
        origEdges.push_back(c);
    }
    LL maxflow = 0;
    dinic.maxflow();
    vector<int> mincut(dinic.mincut());
    for (int x: mincut) {
        maxflow += origEdges[x];
    }
    fout << maxflow << ' ' << mincut.size() << endl;
    for (int x: mincut)
        fout << x + 1 << endl;
    return 0;
}
```