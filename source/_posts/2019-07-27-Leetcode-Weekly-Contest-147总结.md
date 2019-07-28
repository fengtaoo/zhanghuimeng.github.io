---
title: Leetcode Weekly Contest 147总结
urlname: leetcode-weekly-contest-147
toc: true
date: 2019-07-27 01:49:10
updated: 2019-07-29 01:56:10
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming, alg:Minmax]
categories: Leetcode
---

这次比赛做得有点费劲……

<!--more-->

## [1137. N-th Tribonacci Number](https://leetcode.com/problems/n-th-tribonacci-number/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms

### 题意

求一个递推数列的第`n`项。

### 分析

因为`n`很小，所以直接线性递推就可以了。（不过不做优化的递归大概还是会爆……）

### 代码

```cpp
class Solution {
public:
    int tribonacci(int n) {
        int t0 = 0, t1 = 1, t2 = 1;
        if (n == 0) return 0;
        if (n <= 2) return 1;
        for (int i = 3; i <= n; i++) {
            int t = t0 + t1 + t2;
            t0 = t1;
            t1 = t2;
            t2 = t;
        }
        return t2;
    }
};
```

## [1138. Alphabet Board Path](https://leetcode.com/problems/alphabet-board-path/description/)

标记难度：Medium

提交次数：1/2

代码效率：4ms

### 题意

有一个方格图是这样的（空的地方没有道路）：

```txt
a b c d e
f g h i j
k l m n o
p q r s t
u v w x y
z
```

从`a`处开始，要求给出一条能够打印出目标字符串的最短路线。

### 分析

这个方格图上最难处理的地方显然是`z`。当然，处理方法实际上很简单：先走到`z`正上方的`u`，再向下一格到`z`；其他字母间的相互移动直接走曼哈顿距离就可以了。

### 代码

```cpp
class Solution {
    // 坐标差对应的路线
    string calc(int dx, int dy) {
        string ans;
        for (int i = 0; i < abs(dx); i++)
            ans += dx < 0 ? 'U' : 'D';
        for (int i = 0; i < abs(dy); i++)
            ans += dy < 0 ? 'L' : 'R';
        return ans;
    }
    
    // 从一个字母到另一个字母的路线
    string calc2(char last, char next) {
        if (last != 'z' && next != 'z')
            return calc((next - 'a') / 5 - (last - 'a') / 5, (next - 'a') % 5 - (last - 'a') % 5);
        if (last == 'z' && next == 'z')
            return "";
        if (last == 'z')
            return 'U' + calc((next - 'a') / 5 - ('u' - 'a') / 5, (next - 'a') % 5 - ('u' - 'a') % 5);
        if (next == 'z')
            return calc(('u' - 'a') / 5 - (last - 'a') / 5, ('u' - 'a') % 5 - (last - 'a') % 5) + 'D';
        return "";
    }
    
public:
    string alphabetBoardPath(string target) {
        string ans;
        for (int i = 0; i < target.length(); i++) {
            ans += i == 0 ? calc2('a', target[i]) : calc2(target[i-1], target[i]);
            ans += '!';
        }
        return ans;
    }
};
```

## [1139. Largest 1-Bordered Square](https://leetcode.com/problems/largest-1-bordered-square/description/)

标记难度：Medium

提交次数：1/1

代码效率：8ms

### 题意

给定一个01矩阵，问其中最大的边框均为1的正方形的面积。

### 分析

直接用两个数组维护每个位置向下和向右的连续1的最大数量，然后在每个位置上枚举全部可能的正方形的变长即可，时间复杂度是`O(N^3)`。似乎是一种平凡的DP的变形。

### 代码

```cpp
class Solution {
public:
    int largest1BorderedSquare(vector<vector<int>>& grid) {
        int maxDown[101][101], maxRight[101][101];
        int n = grid.size(), m = grid[0].size();
        memset(maxDown, 0, sizeof(maxDown));
        memset(maxRight, 0, sizeof(maxRight));
        for (int i = n - 1; i >= 0; i--) {
            for (int j = m - 1; j >= 0; j--) {
                if (grid[i][j] == 0) {
                    maxDown[i][j] = maxRight[i][j] = 0;
                    continue;
                }
                maxDown[i][j] = i == n - 1 ? 1 : 1 + maxDown[i+1][j];
                maxRight[i][j] = j == m - 1 ? 1 : 1 + maxRight[i][j+1];
            }
        }
        
        int ans = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] == 0) continue;
                for (int l = ans + 1; i+l <= n && maxRight[i][j] >= l && j + l <= m; l++) {
                    if (maxDown[i][j] >= l && maxRight[i][j] >= l && maxDown[i][j+l-1] >= l && maxRight[i+l-1][j] >= l)
                        ans = max(ans, l);
                }
            }
        }
        return ans * ans;
    }
};
```

## [1140. Stone Game II](https://leetcode.com/problems/stone-game-ii/description/)

标记难度：Medium

提交次数：1/1

代码效率：0ms

### 题意

两个玩家对一列石子堆轮流进行操作：初始时`M = 1`，每次一个人可以取最前面的`X`堆石子，其中`X`满足`1 <= X <= 2*M`，然后令`M = max(M, X)`；之后换另一个人操作。问两人都以最佳策略操作时，第一个人最多可以得到多少石子？

### 分析

这显然是个MinMax问题，但是我最开始的时候把状态转移方向弄反了，搞成了从后向前转移；实际上应该是从前向后。（显然，状态转移方向和状态的数字大小没有必然联系。）除此之外，这道题还是很简单的。

（专门看了一下，[877. Stone Game](https://leetcode.com/problems/stone-game/description/)的确也是Alex和Lee玩游戏……）

### 代码

```cpp
class Solution {
    int f[102][102][2];  // i, M, player0/1
    int sum[102][102];
    int n;
    
    void calc(int i, int M) {
        if (f[i][M][0] != -1) return;
        f[i][M][0] = f[i][M][1] = 0;
        for (int X = 1; X <= 2 * M; X++) {
            if (i + X > n) break;
            int i1 = i + X, M1 = max(X, M);
            calc(i1, M1);
            if (sum[i][i1-1] + f[i1][M1][1] > f[i][M][0]) {
                f[i][M][0] = sum[i][i1-1] + f[i1][M1][1];
                f[i][M][1] = f[i1][M1][0];
            }
        }
    }
    
public:
    int stoneGameII(vector<int>& piles) {
        n = piles.size();
        for (int i = 0; i < n; i++) {
            sum[i][i] = piles[i];
            for (int j = i + 1; j < n; j++)
                sum[i][j] = sum[i][j-1] + piles[j];
        }
        memset(f, -1, sizeof(f));
        calc(0, 1);
        
        return f[0][1][0];
    }
};
```
