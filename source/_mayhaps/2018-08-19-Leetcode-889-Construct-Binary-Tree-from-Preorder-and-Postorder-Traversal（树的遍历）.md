---
title: 'Leetcode 889. Construct Binary Tree from Preorder and Postorder Traversal（树的遍历）'
urlname: leetcode-889-construct-binary-tree-from-preorder-and-postorder-traversal
toc: true
date: 2018-08-19 20:41:07
updated: 2018-08-19 20:41:07
tags: [Leetcode, LeetcodeContest]
---

题目来源：[https://leetcode.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/description/](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/description/)

标记难度：Medium

提交次数：

代码效率：

* 递归版本：8ms
* 迭代版本：

## 题意

给定一棵二叉树的先序遍历和后序遍历，返回任意一棵满足条件的二叉树。

## 分析

这道题是周赛的第二题。我在20分钟时提交了，所以加上中间急眼去查资料的时间，只写了9分钟？真是令人惊讶……

我已经听人说过这个结论上百次了：先序+中序可以重建，中序+后序可以重建，先序+后序不能重建……所以看到这个问题的时候我非常懵逼。但事实上，在规定一些条件的情况下——比如要求重建出来的树必须是完全二叉树——完全可以建出一棵符合要求的树来。[^geek]

[^geek]: [Construct Full Binary Tree from given preorder and postorder traversals](https://www.geeksforgeeks.org/full-and-complete-binary-tree-from-given-preorder-and-postorder-traversals/)

迭代的思路是非常好想的。给定一棵树的先序和后序遍历结果，我们显然可以得知，树根在先序遍历的开头和后序遍历的结尾。接下来只要把剩余的内容切分成左子树和右子树，就可以开始递归了。但问题是如何切分。事实上我们只能保证树根之后的第一个元素是它的一个非空子节点，但却不能知道这个元素是它的左子节点还是右子节点。这就是先序+后序遍历多种可能性的来源了。

```
[root][......left......][...right..]  ---pre
[......left......][...right..][root]  ---post
```

下一个问题是，给定先序遍历和后序遍历，能否构造出满足条件的所有二叉树，或者仅仅统计它们的数量？我总觉得也许曾经做过这样的题，但现在暂时想不动了。

### 迭代算法


## 代码

### 递归版本

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
private:
    vector<int> pre, post;
    TreeNode* constructTree(int preL, int preR, int postL, int postR) {
        // cout << preL << ' ' << preR << ' ' << postL << ' ' << postR << endl;
        if (preL == preR)
            return new TreeNode(pre[preL]);
        if (pre[preL] != post[postR])
            return NULL;
        int i = postL;
        while (i <= postR && post[i] != pre[preL + 1]) {
            i++;
        }
        // [postL, i] is the left subtree
        int j = preL + i - postL + 1;
        // [preL+1, j] is the left subtree
        TreeNode* root = new TreeNode(pre[preL]);
        root->left = constructTree(preL+1, j, postL, i);
        root->right = constructTree(j+1, preR, i+1, postR-1);
        return root;
    }

public:
    TreeNode* constructFromPrePost(vector<int>& pre, vector<int>& post) {
        // for full trees, this is really feasible!
        // https://www.geeksforgeeks.org/full-and-complete-binary-tree-from-given-preorder-and-postorder-traversals/
        if (pre.size() == 0)
            return NULL;
        if (pre.size() != post.size())
            return NULL;

        this->pre = pre;
        this->post = post;
        return constructTree(0, pre.size() - 1, 0, post.size() - 1);
    }
};
```
