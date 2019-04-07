---
title: Leetcode 589. N-ary Tree Preorder Traversal（树）
urlname: leetcode-589-n-ary-tree-preorder-traversal
toc: true
date: 2018-08-01 19:15:55
updated: 2018-08-01 19:15:55
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/n-ary-tree-preorder-traversal/description/](https://leetcode.com/problems/n-ary-tree-preorder-traversal/description/)

标记难度：Easy

提交次数：1/1

代码效率：98.78%

## 题意

N叉树先序遍历。

## 分析

这道题从某种意义上可以说是上回做的[Leetcode 173](/post/leetcode-173-binary-search-tree-iterator)的升级版，这回变成了多叉树先序遍历。如果直接用递归的方法来做，那这就是一个超级大水题。

### 迭代版多叉树先序遍历

多叉树的先序遍历实质上就是按顺序对子树进行递归先序遍历，所以为了实现迭代，我们仍然可以像之前那样借用栈存储尚未访问到的子树。下面的代码摘自[C++ simple 10-line iterative solution, beat 100%!](https://leetcode.com/problems/n-ary-tree-preorder-traversal/discuss/150297/C++-simple-10-line-iterative-solution-beat-100!)

```
vector<int> preorder(Node* root) {
    if(root==NULL) return {};
    stack<Node*> stk;
    vector<int> res;
    stk.push(root);
    while(!stk.empty()) {
        Node* temp=stk.top();
        stk.pop();
        for(int i=temp->children.size()-1;i>=0;i--) stk.push(temp->children[i]);
        res.push_back(temp->val);
    }
    return res;
}
```

### 多叉树遍历与二叉树遍历的关系

## 代码
