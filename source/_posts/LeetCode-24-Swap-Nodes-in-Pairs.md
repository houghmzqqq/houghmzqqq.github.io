---
title: 'LeetCode:24.Swap Nodes in Pairs'
date: 2017-09-01 20:51:35
tags:
    - LeetCode-频度5
    - 算法
    - Medium
---


## 思路
原题：Given a linked list, swap every two adjacent nodes and return its head.
For example,

Given 1->2->3->4, you should return the list as 2->1->4->3.

Your algorithm should use only constant space. You may not modify the values in the list, only nodes itself can be changed.

大意：给定一个链表，将相邻的两个节点互换。只能够使用恒定的物理空间，不能够改变节点的值，只能够改变节点本身。

<!-- more -->
分析：这道题中需要定义1个辅助变量root。

## 代码
```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode root = new ListNode(-1);  
        root.next = head;  
        ListNode pre_head = root;  
        while(head != null && head.next!= null){  
            ListNode Node1 = head;  
            ListNode Node2 = head.next.next;  
            root.next = head.next;
			//改变单数节点
            root = root.next;
			//改变双数节点
            root.next = Node1;
			//跳转到下一对要置换的节点
            Node1.next = Node2;
            root = root.next;
            head = Node2;
              
        }  
        return pre_head.next; 
    }
}
```
