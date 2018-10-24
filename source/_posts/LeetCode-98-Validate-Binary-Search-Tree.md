---
title: 'LeetCode:98.Validate Binary Search Tree'
date: 2017-08-14 13:28:14
tags:
	- LeetCode-频度5
	- 算法
	- Medium
---


## 思路
原题：Given a binary tree, determine if it is a valid binary search tree (BST).
Assume a BST is defined as follows:
>	The left subtree of a node contains only nodes with keys less than the node's key.
>	The right subtree of a node contains only nodes with keys greater than the node's key.
>	Both the left and right subtrees must also be binary search trees.

<!-- more -->
大意：判断一个二叉树是不是二叉搜索树（BST）。
二叉搜索树的定义：
>	左边的节点小于父节点
>	右边的节点大于父节点
>	子树也是二叉搜索树

分析：因为一个节点的子节点需要与该节点的父节点相比较，所以这里定义两个Integer类型的属性min和max，分别用作和左子节点和右子节点作比较。然后用一个递归 算法遍历一遍二叉树。

## 代码
```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    
    Map<TreeNode,TreeNode> map = new HashMap();
    
    public boolean isValidBST(TreeNode root) {
        
        return isValid(root,null,null);
        
    }
    
    public boolean isValid(TreeNode root,Integer min,Integer max){
        if(root==null) return true;
        if(min!=null && root.val<=min) return false;
        if(max!=null && root.val>=max) return false;
        
        return isValid(root.left,min,root.val) && isValid(root.right,root.val,max);
    }
}
```