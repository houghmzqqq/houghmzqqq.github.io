---
title: 'LeetCode:77.Combinations'
date: 2017-09-02 16:01:58
tags:
	- LeetCode-频度5
	- 算法
	- Medium
	- 递归
	- 排列组合

---


## 思路
原题：Given two integers n and k, return all possible combinations of k numbers out of 1 ... n.

For example,
If n = 4 and k = 2, a solution is:
> [
>  [2,4],
>  [3,4],
>  [2,3],
>  [1,2],
>  [1,3],
>  [1,4],
> ]

大意：给定两个数字n和k，返回1...n中k个数字的所有排列组合结果。

<!-- more -->
分析：通过递归创建分支：1.当长度为k时，加入结果集；2.当剩下的元素与当前的元素相加小于k时，跳出该分支。
（与[LeetCode49题](https://houghmzqqq.github.io/2017/09/01/LeetCode-49-Group-Anagrams/)类似）(78题也与本题相似)


## 代码
```
class Solution {
    public List<List<Integer>> combine(int n, int k) {
        List<Integer> midList = new ArrayList();
        List<List<Integer>> res = new ArrayList();
        
        group(midList,res,n,k,1);
        
        return res;
    }
    
    public void group(List<Integer> midList,List<List<Integer>> res,int n,int k,int x){
        //List<Integer> ns = new ArrayList(nList);
        List<Integer> l1 = new ArrayList(midList);
        
		//长度等于k是，加入结果集
        if(l1.size()==k){
            res.add(l1);
            return ;
        }
        if(l1.size()+n-x+1<k){
            return ;
        }
        
        for(int i=x;i<=n;i++){
			//创建分之后回溯
            l1 = new ArrayList(midList);
			
			//创建分支
            l1.add(i);
            group(l1,res,n,k,i+1);
        }
    }
}
```