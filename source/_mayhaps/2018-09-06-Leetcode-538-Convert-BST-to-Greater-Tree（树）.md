---
title: Leetcode 538. Convert BST to Greater Tree（树）
urlname: leetcode-538-convert-bst-to-greater-tree
toc: true
date: 2018-09-06 16:33:59
updated: 2018-09-06 16:33:59
tags: [Leetcode, alg:Tree, alg:In-Order Traversal]
---

题目来源：[https://leetcode.com/problems/convert-bst-to-greater-tree/description/](https://leetcode.com/problems/convert-bst-to-greater-tree/description/)

标记难度：Easy

提交次数：1/1

代码效率：

* 两遍DFS：17.19%
* 一遍DFS：36.85%

## 题意

给定一棵BST，在它的每个结点上加上之前比它大的所有结点的值。

## 分析

### DFS

通过两遍DFS很好解决：首先计算出树上所有结点的和，然后进行中序遍历，对于每个结点，先把当前的和传递给左子树，然后把自己的值从这个和例减去，再把和传递给右子树。

通过一遍DFS也很好解决。我使用了一种类似于递推的方法：`int dfs(TreeNode* root, int down)`，其中`down`是需要加到`root`及其子树的所有结点上的值，函数的返回值是`root`及其所有子树原来的值之和。首先计算出右子树的值的和`rightSum = dfs(root->right, down)`，然后计算出左子树的值的和（显然需要加上右子树和`root`的值）`leftSum = dfs(root->left, down + rightSum + root->val)`，最后更新`root`的值，并返回`leftSum + rightSum + root->val (old)`。

当然这种方法有点太繁琐了，用一个全局变量记录比当前结点大的所有结点的值之和，然后从右子树开始遍历更容易一些。[^recursive]

[^recursive]: [Approach \#1 Recursion \[Accepted\]](https://leetcode.com/articles/convert-bst-to-greater-tree/#approach-1-recursion-accepted)

### 栈

这种“右-中-左”的访问方法大概可以叫“逆中序遍历”。迭代版“逆中序遍历”的实现方法和思路和中序遍历很类似：沿着最右侧通路，自底而上依次访问沿途各节点及其左子树。因此可以参照中序遍历的代码进行一点微小的修改[^dsapp]：

```cpp
// 代码5.15* 二叉树逆中序遍历算法（迭代版#1）
template <typename T> //从当前节点出发，沿右分支不断深入，直至没有右分支的节点
static void goAlongRightBranch(BinNodePosi(T) x, Stack<BinNodePosi(T)>& S) {
    while (x) { S.push(x); x = x->rChild; } //当前节点入栈后随即向右侧分支深入，迭代直到无右孩子
}

template <typename T, typename VST> //元素类型、操作器
void travIn_I1(BinNodePosi(T) x, VST& visit) { //二叉树逆中序遍历算法（迭代版#1）
    Stack<BinNodePosi(T)> S; //辅助栈
    while (true) {
        goAlongRightBranch(x, S); //从当前节点出发，逐批入栈
        if (S.empty()) break; //直至所有节点处理完毕
        x = S.pop(); visit(x->data); //弹出栈顶节点并访问之
        x = x->lChild; //转向左子树
    }
}
```

[^dsapp]: 《数据结构(C++语言版)》（第三版） P129，清华大学出版社，2013.9

以及我们之前好像在[Leetcode 173](/post/leetcode-173-binary-search-tree-iterator)里讨论过中序遍历中找后继的问题了。

### Morris中序遍历

这是一种利用树上的空指针，在常数空间下进行中序遍历的方法。（下列描述针对的是中序遍历；对于逆中序遍历，反过来就可以了。）

* 令`p = root`
* 进行下列操作直到`p == null`：
  * 如果`p`没有左子树，则访问`p->val`；`p = p->right`
  * 找到`p`的直接前驱结点`prev`（通过向左走一步后不断向右走）
    * 如果`prev->right == p`，说明左子树已经访问完成，令`prev->right = null`，访问`p->val`，`p = p->right`
    * 否则令`prev->right = p`，`p = p->left`，开始访问左子树

## 代码

### 两遍DFS

```cpp
class Solution {
private:
    int sum;

    void addSum(TreeNode* root) {
        if (root == NULL) return;
        sum += root->val;
        addSum(root->left);
        addSum(root->right);
    }

    void addNode(TreeNode* root) {
        if (root == NULL) return;
        addNode(root->left);
        sum -= root->val;
        root->val += sum;
        addNode(root->right);
    }

public:
    TreeNode* convertBST(TreeNode* root) {
        sum = 0;
        addSum(root);
        addNode(root);
        return root;
    }
};
```

### 一遍DFS

```cpp
class Solution {
private:
    // 返回root子树的和
    // down：这个值需要加到所有子树上
    int dfs(TreeNode* root, int down) {
        if (root == nullptr) return 0;
        int rightSum = dfs(root->right, down);
        int leftSum = dfs(root->left, down + rightSum + root->val);
        int sum = leftSum + rightSum + root->val;
        root->val += down + rightSum;
        return sum;
    }

public:
    TreeNode* convertBST(TreeNode* root) {
        dfs(root, 0);
        return root;
    }
};
```

### 栈

```cpp
class Solution {
public:
    TreeNode* convertBST(TreeNode* root) {
        int sum = 0;
        TreeNode* p = root;
        stack<TreeNode*> s;
        while (true) {
            while (p != nullptr) {
                s.push(p);
                p = p->right;
            }
            if (s.empty()) break;
            p = s.top();
            s.pop();
            p->val += sum;
            sum = p->val;
            p = p->left;
        }
        return root;
    }
};
```
