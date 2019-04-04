---
title: Leetcode 478. Generate Random Point in a Circle（随机）
urlname: leetcode-478-generate-random-point-in-a-circle
toc: true
mathjax: true
date: 2018-08-14 17:20:01
updated: 2018-08-14 22:17:01
tags: [Leetcode, alg:Math, alg:Random, alg:Rejection Sampling]
---

题目来源：[https://leetcode.com/problems/generate-random-point-in-a-circle/description/](https://leetcode.com/problems/generate-random-point-in-a-circle/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* 极坐标法：64.06%
* 拒绝取样：97.20%

## 题意

给定平面上一个圆的圆心位置和半径，从圆中以**均匀**的概率随机选取点。

## 分析

### 拒绝取样

其实我的第一反应是用拒绝取样（[Rejection Sampling](https://en.wikipedia.org/wiki/Rejection_sampling)）的思路来做：首先从这个圆的与坐标轴平行的外切正方形中均匀随机选取点，然后判断点是否位于圆中；如果不在，重新生成一个新的点，再次进行判断；否则直接返回。

直觉上来说，拒绝取样显然是正确的；不过我们可以用一种稍微更加形式化的方法来描述。（以下内容参考了[拒绝采样（reject sampling）的简单认识](https://gaolei786.github.io/statistics/reject.html)，非常直观形象。）

>下图是一个随机变量的密度函数曲线，试问如何获得这个随机变量的样本呢？

如果你像我一样，已经把概率论与数理统计统统还给数学老师了，那么提示一下，概率密度函数（PDF）是累积分布函数（CDF）的导数，反映的是概率的“密集程度”。你可以设想一根极细的无穷长的金属杆，概率密度相当于杆上各点的质量密度。由于上述原因，概率密度函数曲线下的面积表示的就是取值概率。（参考了[应该如何理解概率分布函数和概率密度函数？](https://www.jianshu.com/p/b570b1ba92bb)）

![概率密度函数曲线](reject1.png)

>我们首先用一个矩形将这个密度曲线套起来，把密度曲线框在一个矩形里，如下图所示：

![被框起来的概率密度函数曲线](reject2.png)

>然后，向这个矩形里随机投点。随机投点意味着在矩形这块区域内，这些点是满足均匀分布的。投了大概10000个点，如下面这个样子：

![投点之后的分布情况](reject3.png)

>显然，有的点落在了密度曲线下侧，有的点落在了密度曲线的上侧。上侧的点用绿色来表示，下侧的点用蓝色来表示，如下图：

![落在上侧和下侧的点](reject4.png)

>只保留密度曲线下侧的点，即蓝色的点：

![保留下侧的点](reject5.png)

>到这里，提一个问题：在密度曲线以下的这块区域里，这些点满足什么分布？均匀分布！这是拒绝采样最关键的部分，搞个矩形、向矩形里投点等等，所做的一切都是为了获得一个密度曲线所围成区域的均匀分布。只要能获得这样一个在密度曲线下满足均匀分布的样本，我们就可以获得与该密度曲线相匹配的随机变量的采样样本。方法是，只需把每个蓝点的横坐标提取出来，这些横坐标所构成的样本就是我们的目标样本。下图左侧，是按照以上方法获得的一个样本的直方图以及核密度估计，下图右侧，是开始的密度曲线。

![统计结果](reject6.png)

[Reject sampling solution](https://leetcode.com/problems/generate-random-point-in-a-circle/discuss/154790/Reject-sampling-solution)中给出了一种简单明了的用拒绝采样实现此题的方法。

### 极坐标法

但是我觉得这个方法的逼格不够高。于是我心想，我们怎么表示一个圆内的点呢？那自然就是用极坐标法，半径+角度。那么我们随机一个半径，再随机一个角度，最后换算成直角坐标系里的坐标，不就可以了嘛！

……只是听上去可以而已……

交上去之后我发现，一共有8个测试样例，最后一个我总是过不去。我的随机性是不是真的写出了bug？我实在是想不出来错在哪里了，于是去看了看题解区——原来有这么多人和我有一样的困惑。是的，上述思路确实有问题。

当我写出上述代码的时候，我心里想的是：先用角度随机选择一条半径，然后在这条半径上随机选择一个点。但问题是我们不能把圆这样看成是很多半径的集合——或者说，这条半径上不同位置的点的密集程度是不一样的。显然距离圆心更远的一端被选择的概率应该更大。

![在圆上随机选择](circle.jpg)

直接对角度和半径都进行随机取样会产生这样的后果，靠近圆心的点被选择的概率偏大（[Uniform random points in a circle using polar coordinates](http://www.anderswallin.net/2009/05/uniform-random-points-in-a-circle-using-polar-coordinates/)）：

![左侧是错误的随机结果，右侧是正确的结果](distribution.jpg)

我们重新考虑一下半径的问题。以半径$r$作为随机变量，则随机点落在$r$范围内的概率分布为

$$\text{CDF}(r) = \frac{r^2}{R^2}$$

$$\text{PDF}(r) = \frac{d}{dr} \text{CDF}(r) = \frac{2r}{R^2}$$

以`[0, 1]`均匀分布对$\text{CDF}(r)$进行随机，则有$r = R \sqrt{\text{rand}()}$。

### C++中的数学运算

使用[std::uniform_real_distribution](http://en.cppreference.com/w/cpp/numeric/random/uniform_real_distribution)生成随机实数比直接用`rand()`好得多，至少[stackoverflow](https://stackoverflow.com/questions/686353/c-random-float-number-generation)上是这么说的。具体使用方法如下：

```cpp
#include <iostream>
#include <iomanip>
#include <string>
#include <map>
#include <random>

int main()
{
    std::random_device rd;

    //
    // Engines
    //
    std::mt19937 e2(rd());
    //std::knuth_b e2(rd());
    //std::default_random_engine e2(rd()) ;

    //
    // Distribtuions
    //
    std::uniform_real_distribution<> dist(0, 10);
    //std::normal_distribution<> dist(2, 2);
    //std::student_t_distribution<> dist(5);
    //std::poisson_distribution<> dist(2);
    //std::extreme_value_distribution<> dist(0,2);

    double rand = dist(e2));

    return 0;
}
```

目前C++中并没有提供什么特别高级的获得`PI`值的方法，仍然是用C中提供的`M_PI`宏：

```cpp
#define _USE_MATH_DEFINES  // 根据实际情况；参见https://stackoverflow.com/questions/1727881/how-to-use-the-pi-constant-in-c
#include <math.h>
M_PI
```

`std::cos`和`std::sin`函数使用的是弧度。好久不写，都忘光了。

## 代码

### 拒绝取样

```cpp
class Solution {
private:
    double radius, x_center, y_center, x_leftdown, y_leftdown;
    std::random_device rd;
    std::mt19937 gen;
    std::uniform_real_distribution<> dis;

public:
    Solution(double radius, double x_center, double y_center): gen(rd()), dis(0, 1) {
        this->radius = radius;
        this->x_center = x_center;
        this->y_center = y_center;
        x_leftdown = x_center - radius;
        y_leftdown = y_center - radius;
    }

    vector<double> randPoint() {
        double x, y;
        do {
            x = x_leftdown + 2 * radius * dis(gen);
            y = y_leftdown + 2 * radius * dis(gen);
        } while ( (x - x_center) * (x - x_center) + (y - y_center) * (y - y_center) > radius * radius);

        return {x, y};
    }
};
```

### 极坐标

```cpp
class Solution {
private:
    double radius, x_center, y_center;
    std::random_device rd;
    std::mt19937 gen;
    std::uniform_real_distribution<> randDeg, randRadius;

public:
    Solution(double radius, double x_center, double y_center): gen(rd()), randDeg(0, 2 * M_PI), randRadius(0, 1) {
        this->radius = radius;
        this->x_center = x_center;
        this->y_center = y_center;
    }

    vector<double> randPoint() {
        double r = sqrt(randRadius(gen)) * radius;  // !
        double deg = randDeg(gen);
        double x = x_center + r * cos(deg);
        double y = y_center + r * sin(deg);

        return {x, y};
    }
};
```
