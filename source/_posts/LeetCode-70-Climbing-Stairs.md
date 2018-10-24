---
title: 'LeetCode:70.Climbing Stairs'
date: 2017-07-28 20:00:07
tags:
    - LeetCode-频度5
    - 算法
    - Easy
---

## 思路
原题：You are climbing a stair case. It takes n steps to reach to the top.

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

<!-- more -->
大意：你去爬一座山，山脚到山顶有n级台阶。没一次你可以爬一或两级台阶。计算到达山顶有几种走法。

分析：假设有n级台阶，有s种走法，那么
当n=1,s=1;
n=2,s=2;
n=3,s=3;
n=4,s=5;
n=5,s=8;
很明显，这是一个裴波那切数列，可以使用递归的方式进行计算，公式为f(n)=f(n-2)+f(n-1).

## 代码
```
public class Solution {
    public int climbStairs(int n) {
        if(n==1 || n==2){
            return n;
        }
        else{
            return climbStairs(n-2)+climbStairs(n-1);
        }
    }
}
```

## 另一种解法
分析：递归是从目标值算到基础值，可以反过来，通过迭代的方式从基础值算到目标值。

## 代码
```
public class Solution {
    public int climbStairs(int n) {
        if(n==1 || n==2){
            return n;
        }
        int n1 = 1;
        int n2 = 2;
        for(int i=3;i<=n;i++){
            int temp = n1+n2;
            n1 = n2;
            n2 = temp;
        }
        return n2;
    }
}
```