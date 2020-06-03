---
title: 二叉搜索树的第k个结点
date: 2018-10-31 00:08:52
categories: algorithm
tags: leetcode
---

### [二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

难度 **中等**



给定一个二叉搜索树，编写一个函数 `kthSmallest` 来查找其中第 **k **个最小的元素。

**说明：**  
你可以假设 k 总是有效的，1 ≤ k ≤ 二叉搜索树元素个数。

**示例 1:**

```
**输入:** root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
**输出:** 1```

**示例 2:**

```
**输入:** root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
**输出:** 3```

**进阶：**  
如果二叉搜索树经常被修改（插入/删除操作）并且你需要频繁地查找第 k 小的值，你将如何优化 `kthSmallest` 函数？

####  解题思路
二叉搜索树 已经就是一个排好序的树了，我们只要按照中序遍历到第k个数，输出第k个数的值即可。

#### Solution

Language: **Java**

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int count = 0;
    public int kthSmallest(TreeNode root, int k) {
         if (root == null || k < 0){
            return 0;
        }
        count = k;
        
        return inOrder(root);
    }
    
    private int inOrder(TreeNode root){
        if(root == null)
            return -1;
        int val = inOrder(root.left);
        if(count == 0)
            return val;
        if(--count == 0)
            return root.val;
        return inOrder(root.right);
    }
    
}
```
或者使用方法二：
```
class Solution {
        private int index=0;
        public int kthSmallest(TreeNode root, int k) {
            if(root==null)
                return Integer.MAX_VALUE;
            int ret=kthSmallest(root.left,k);
            if(ret!=Integer.MAX_VALUE)
                return ret;
            index++;
            if(index==k)
            {
                return root.val;
            }
            return  kthSmallest(root.right,k);
        }
    }
```