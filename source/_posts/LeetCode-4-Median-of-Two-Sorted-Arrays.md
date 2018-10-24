---
title: 'LeetCode:4.Median of Two Sorted Arrays'
date: 2017-07-17 16:20:28
tags: 
	- LeetCode-频度5
	- 算法
	- Hard
---


## 思路
原题：There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

<!-- more -->
大意：给出两个已经排好序的int数组，将其合并，获取中数并返回。

分析：要求获取两个数组合并后的中数。首先要进行的是对两个数组的合并，然后对合并后的数组进行排序，由于给出的数组本身已经排序，所以可以很容易地在合并的过程中进行排序，最后取出中数加以计算即可。

步骤：1、将两个数组合并，合并过程中进行排序 2、计算中数

## 代码

```
public class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
       int[] numAccount = new int[nums1.length+nums2.length];
		
		//两个int数组合并为一个，并排序
		int i=0;
		int j=0;
		for(int z=0;z<numAccount.length;z++)
		{
			//判断是否超出数组范围
			if(i >= nums1.length)
			{
				numAccount[z] = nums2[j];
				j++;
			}
			else if(j >= nums2.length)
			{
				numAccount[z] = nums1[i];
				i++;
			}
			//先比较大小再填入numAccount数组中
			else if(nums1[i]<=nums2[j])
			{
				numAccount[z] = nums1[i];
				i++;
			}
			else if(nums2[j]<nums1[i])
			{
				numAccount[z] = nums2[j];
				j++;
			}
			
		}
		
		//取出中数
		if(numAccount.length%2 == 1)
		{
			return numAccount[numAccount.length/2];
		}
		else
		{
			int midNum1,midNum2;
			midNum1 = numAccount[(numAccount.length/2) - 1];
			midNum2 = numAccount[numAccount.length/2];
			
			return (double)(midNum1 + midNum2) /2;
		}
    }
}
```
