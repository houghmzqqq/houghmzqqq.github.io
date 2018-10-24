---
title: 'LeetCode:15.3Sum'
date: 2017-07-22 21:53:10
tags:
	- LeetCode-频度5
	- 算法
	- Medium
---


## 思路
原题：Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.
__Note:__ The solution set must not contain duplicate triplets.

<!-- more -->
大意：给一个包含n个整数的数组，找出所有符合a+b+c=0的元素数组。
__提示__：a,b,c符合条件a<=b<=c，结果不能重复。

初次分析：题目要求结果要排序并且不能重复，所以可以先对数组进行排序，在通过三重for循环得出结果，因为需要查重，所以需要四重for循环。

步骤（1）：
	1、对数组进行排序
	2、三重循环遍历所有可能的元素
	3、得到结果后，对比是否重复
	4、结果放入List中

## 代码（1）：
```
public class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<Integer> sum = null;
        List<List<Integer>> sums = new ArrayList<List<Integer>>();
        
        for(int i=nums.length-1;i>0;i--)
        {
            for(int j=0;j<i;j++)
            {
                if(nums[j+1]<nums[j])
                {
                    int temp;
                    temp = nums[j];
                    nums[j] = nums[j+1];
                    nums[j+1] = temp;
                }
            }
        }
        
        for(int x=0;x<nums.length-2;x++)
        {
            
            for(int y=x+1;y<nums.length-1;y++)
            {
                for(int z=y+1;z<nums.length;z++)
                {
                    if(nums[x]+nums[y]+nums[z] == 0)
                    {
                        sum = new ArrayList<Integer>();
                        boolean repeat = false;
                        
                        sum.add(nums[x]);
                        sum.add(nums[y]);
                        sum.add(nums[z]);
                        
                        for(List<Integer> comp : sums)
                        {
                           if(sum.get(0)==comp.get(0) && sum.get(1)==comp.get(1) && sum.get(2)==comp.get(2))
                           {
                               repeat = true;
                           }
                        }
                        
                        if(!repeat)
                        {
                            sums.add(sum);
                        }
                    }
                }
            }
        }
        return sums;
    }
}
```
这个答案submit后出错，提示：Time Limit Exceeds(超时)。
经分析，是因为套入多重的for循环，时间复杂度大，当数组的基数较大时，运行时间会超出限制。

## 修改
解决方案：减少for循环的层数。通过查找资料了解到，可以借助map避免第三层循环。

步骤（2）：
	1、对数组进行排序
	2、定义一个map，以nums（给定数组）的值为 __key__ ，以list（该值在数组中的index）为__value__。
	3、遍历数组，排除明显不符合条件的结果，并得出符合条件的结果。
	4、结果放入List中

## 代码（2）
```
public class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<Integer> sum = null;
        List<List<Integer>> sums = new ArrayList<List<Integer>>();
        
        //排序
        for(int i=nums.length-1;i>0;i--)
        {
            for(int j=0;j<i;j++)
            {
                if(nums[j+1]<nums[j])
                {
                    int temp;
                    temp = nums[j];
                    nums[j] = nums[j+1];
                    nums[j+1] = temp;
                }
            }
        }
        
        //nums数组中的数字分组放入map中，并记录它们的index(改操作可以减少一重循环)
        Map<Integer,List<Integer>> map = new HashMap();
        for(int i=0;i<nums.length;i++)
        {
            int num = nums[i];
            if(map.get(num)==null)
            {
                List<Integer> lastOnes = new ArrayList();
                lastOnes.add(i);
                map.put(num,lastOnes);
            }
            else
            {
                map.get(num).add(i);
            }
        }
        
        
        for(int y=0;y<nums.length-2;y++)
        {
            //因为已排序，当第一个数大于0时，不能满足a+b+c=0
            if(nums[y]>0)
            {
                break;
            }
            //nums[y]==nums[y-1]说明num[y]之前已经进行过判断,跳过
            if(y>0 && nums[y] == nums[y-1])
            {
                continue;
            }
            for(int z=y+1;z<nums.length-1;z++)
            {
                //与上同理
                if(z>y+1 && nums[z]==nums[z-1])
                {
                    continue;
                }
                //计算数字c
                int finalNum = -nums[y] -nums[z];
                //c为abc中最后一个数字，所以c >= b
                if(finalNum < nums[z])
                {
                    break;
                }
                
                //查找nums中值为finalNum的List，不存在则跳过
                List<Integer> lastOnes = map.get(finalNum);
                if(lastOnes == null)
                {
                    continue;
                }
                for(Integer lastOne : lastOnes)
                {
                    if(lastOne>z)
                    {
                        sum = new ArrayList();
                        sum.add(nums[y]);
                        sum.add(nums[z]);
                        sum.add(nums[lastOne]);
                        sums.add(sum);
                        break;
                    }
                }
            }
        }
        return sums;
    }
}
```


