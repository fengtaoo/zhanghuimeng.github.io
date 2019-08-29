---
title: Leetcode Weekly Contest 151总结
urlname: 2019-08-26-Leetcode Weekly Contest 151总结
toc: true
date: 2019-08-26 07:30:32
updated: 2019-08-26 07:30:32
tags: [Leetcode, Leetcode Contest, alg:Linked List, alg:Map]
categories: Leetcode
---

这场比赛让人感觉一言难尽，第一题简直没有让人写下去的欲望……但是最后一题出得很好，是道不错的数据结构题。

<!--more-->

## [1169. Invalid Transactions](https://leetcode.com/problems/invalid-transactions/description/)

标记难度：Easy

提交次数：1/1

代码效率：52ms

### 题意

简直让人没有复述的欲望……

大概就是让你在某种意义上去重吧。

### 分析

没有任何意义的题……通过暴搜枚举即可。主要是数据呈现的方式比较恶心。

### 代码

吓得我写了一个Python程序。

```py
class Solution(object):
    def invalidTransactions(self, transactions):
        """
        :type transactions: List[str]
        :rtype: List[str]
        """
        ans = []
        transMap = {}
        for t in transactions:
            l = t.split(',')
            name, time, amount, city = l[0], int(l[1]), int(l[2]), l[3]
            if not name in transMap.keys():
                transMap[name] = []
            transMap[name].append((time, city))
        
        for t in transactions:
            l = t.split(',')
            name, time, amount, city = l[0], int(l[1]), int(l[2]), l[3]
            if amount > 1000:
                ans.append(t)
                continue
            for tup in transMap[name]:
                if abs(time - tup[0]) <= 60 and city != tup[1]:
                    ans.append(t)
                    break
        
        return ans
```

## [1170. Compare Strings by Frequency of the Smallest Character](https://leetcode.com/problems/compare-strings-by-frequency-of-the-smallest-character/description/)

标记难度：Easy

提交次数：1/1

代码效率：44ms

### 题意

定义`f(s)`是字符串`s`中最小的字母的出现次数，对于给定的query和word数组，问对于每个query，共有几个word满足`f(query) < f(word)`。

### 分析

以这道题的数据量，直接把每对query和word枚举一遍即可。只要看懂题意，`f(s)`的比较只是正整数的比较而已。

### 代码

```cpp
class Solution {
public:
    vector<int> numSmallerByLeadingCount(vector<string>& queries, vector<string>& words) {
        vector<int> qSize, wSize;
        for (string word: queries) {
            sort(word.begin(), word.end());
            int cnt = 1;
            for (int i = 1; i < word.length(); i++)
                if (word[i] != word[0]) break;
                else cnt++;
            qSize.push_back(cnt);
        }
        for (string word: words) {
            sort(word.begin(), word.end());
            int cnt = 1;
            for (int i = 1; i < word.length(); i++)
                if (word[i] != word[0]) break;
                else cnt++;
            wSize.push_back(cnt);
        }
        
        sort(wSize.begin(), wSize.end());
        vector<int> ans;
        for (int s: qSize) {
            int cnt = 0;
            for (int s1: wSize)
                if (s < s1)
                    cnt++;
            ans.push_back(cnt);
        }
        return ans;
    }
};
```

## [1171. Remove Zero Sum Consecutive Nodes from Linked List](https://leetcode.com/problems/remove-zero-sum-consecutive-nodes-from-linked-list/description/)

标记难度：Medium

提交次数：1/1

代码效率：28ms

### 题意

从一个链表中移除所有和为0的连续结点。可能有多解。

### 分析

直接通过前缀和贪心地移除所有和为0的连续结点即可。我写得过于麻烦了，先把每个结点都记录下来，然后算前缀和，标记删除区间，最后再重建链表。更简单的写法可以参考[^lee]。

[^lee]: [lee215's Solution for Leetcode 1171 - \[Java/C++/Python\] Greedily Skip with HashMap](https://leetcode.com/problems/remove-zero-sum-consecutive-nodes-from-linked-list/discuss/366319/JavaC++Python-Greedily-Skip-with-HashMap)

### 代码

```cpp
class Solution {
public:
    ListNode* removeZeroSumSublists(ListNode* head) {
        vector<ListNode*> lists;
        vector<int> val;
        vector<int> preSum;

        // 记录每个结点
        for (ListNode* p = head; p != NULL; p = p->next) {
            lists.push_back(p);
            val.push_back(p->val);
            if (preSum.size() == 0) preSum.push_back(p->val);
            else preSum.push_back(p->val + preSum.back());
        }

        // 标注最大的可删除区间
        int* flagged = new int[1005];
        for (int i = 0; i < 1005; i++)
            flagged[i] = 0;
        flagged = flagged + 1;
        int n = val.size();
        map<int, vector<int>> indices;
        indices[0].push_back(-1);
        for (int i = 0; i < n; i++) {
            // can delete that
            if (indices[preSum[i]].size() > 0) {
                int first = indices[preSum[i]].front();
                flagged[first]++;
                flagged[i]--;
            }
            indices[preSum[i]].push_back(i);
        }

        // 删除区间和重建
        ListNode* last = NULL;
        head = NULL;
        int sum = flagged[-1];
        for (int i = 0; i < n; i++) {
            if (sum == 0) {
                if (last == NULL)
                    head = last = lists[i];
                else {
                    last->next = lists[i];
                    last = lists[i];
                }
                last->next = NULL;
            }
            sum += flagged[i];
        }
        
        return head;
    }
};
```

## [1172. Dinner Plate Stacks](https://leetcode.com/problems/dinner-plate-stacks/description/)

标记难度：Hard

提交次数：1/1

代码效率：80.26%（572ms）

### 题意

有很多栈排在一起，要求实现以下操作：

* `push`：向最左侧的大小未超过`capacity`的栈顶插入一个元素
* `pop`：从最右侧的非空栈中弹出一个元素
* `popAtStack`：从某一给定`index`的栈中弹出元素

### 分析

这道题的解法大同小异，我一共见到了三个变种：map+set法，set+vector法，以及map+指针法。

其中我觉得set+vector法最好理解。用一个vector维护从0开始到最后一个有元素的栈（因为题目没有给出栈的范围！），并且用一个set维护目前available的栈（在上述vector空间内的）。具体操作步骤参见代码。我写代码的就是这个版本。当然，因为这道题出的性质的原因，这种写法的很大的一个问题被规避了：如果有一种操作是`pushAtStack`，且有一组测试数据是不停地往右侧很大的一个`index`处插入数据再删除，这种方法就得不停地新建栈再删除了。但是并没有这样的操作，所以右侧的变动不会太大。[^vortrubac]

[^vortrubac]: [vortrubac's Solution for Leetcode 1172 - C++ Set, Track Available Stacks](https://leetcode.com/problems/dinner-plate-stacks/discuss/366318/C++-Set-Track-Available-Stacks)

我见到的map+指针法是直接用一个map来存储`index`和栈的对应，然后用两个指针分别标记合法栈的起止位置（即左侧可push位置和右侧可pop位置）。后一个指针的用法非常类似于vector尾部的新建和弹出；当然，考虑到这是一个map，可以做得更好一些，直接查找map尾部的值即可。[^xiao]

[^xiao]: [Java straightforward, HashMap + 2 Pointers](https://leetcode.com/problems/dinner-plate-stacks/discuss/366368/Java-straightforward-HashMap-+-2-Pointers)

map+set法则是把vector换成map，相当于一个map维护所有栈，一个set维护所有available的栈（在当前所有栈范围内的）。这样vector尾部的删除和插入操作就可以省略了。[^leemap]

[^leemap]: [\[C++/Python\] Solution - Solution 2](https://leetcode.com/problems/dinner-plate-stacks/discuss/366331/C++Python-Solution)

### 代码

```cpp
// https://leetcode.com/problems/dinner-plate-stacks/discuss/366331/C++-Map-Solution
class DinnerPlates {
    set<int> available;
    int capacity;
    vector<vector<int>> stks;
    
public:
    DinnerPlates(int capacity) {
        this->capacity = capacity;
    }
    
    void push(int val) {
        if (available.empty()) {
            // 在后面新开一个栈
            available.insert(stks.size());
            stks.push_back(vector<int>());
        }
        // 插入到左边第一个有空间的栈中
        int idx = *(available.begin());
        // 维护vector（因为尾部可能因为空而被删掉了）
        while (stks.size() <= idx)
            stks.push_back(vector<int>());
        stks[idx].push_back(val);
        if (stks[idx].size() >= capacity)
            available.erase(idx);
    }
    
    int pop() {
        // 直接利用popAtStack的实现，弹出最末一个栈的元素
        if (stks.size() == 0) return -1;
        return popAtStack(stks.size() - 1);
    }
    
    int popAtStack(int index) {
        if (index >= stks.size() || stks[index].empty())
            return -1;
        int val = stks[index].back();
        stks[index].pop_back();
        // 删掉vector尾部所有为空的栈
        if (index == stks.size() - 1 && stks[index].empty()) {
            while (stks.size() > 0 && stks.back().empty()) {
                stks.pop_back();
            }
        }
        else
            available.insert(index);
        return val;
    }
};
```
