---
title: 'LeetCode:46.Permutations'
date: 2017-09-01 22:58:10
tags:
	- LeetCode-频度5
	- 算法
	- Medium

---


## 思路
原题：Given a collection of distinct numbers, return all possible permutations.

大意：给定一个数字集合，求所有排列组合的集合

<!-- more -->
分析：以集合[1,2,3,4]为例，分别以1,2,3,4,为开头建立分支，在1为开头的分支中，分别以2,3,4建立分支，以此类推。
使用递归的方式创建分支。

## 代码
```
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> res = new ArrayList();
        List<Integer> firstList = new ArrayList();
        List<Integer> midList = new ArrayList();
        for(int num : nums){
            firstList.add(num);
        }
        
        getList(firstList,midList,res);
        
        return res;
    }
    
	//递归方法
    public void getList(List<Integer> l1,List<Integer> l2,List<List<Integer>> res){
        List<Integer> midList = new ArrayList(l2);
        List<Integer> list = new ArrayList();
        
        if(l1.size()!=0 && l1!=null){
            
            for(int i=0;i<l1.size();i++){
				//没创建一个分支，进行回溯，保证list和midList中个数不变
                list = new ArrayList(l1);
                midList = new ArrayList(l2);
                
                midList.add(list.get(i));
                list.remove(i);
				//创建分支
                getList(list,midList,res);
            }
        }
		//返回条件
        if(l1.size()==0 || l1==null)
            res.add(midList);
    }
    
}
```
