---
title: 'LeetCode:27.Remove Element'
date: 2017-09-01 21:25:51
tags:
	- LeetCode-频度5
	- 算法
	- Easy

---

## 思路
原题：Given an array and a value, remove all instances of that value in place and return the new length.

Do not allocate extra space for another array, you must do this in place with constant memory.

The order of elements can be changed. It doesn't matter what you leave beyond the new length.

大意：给定一个数组和一个常数，删除该数组中所有和该常数相等的值，输出剩余的值。结果返回一个常数，表示结果数组的长度。

<!-- more -->
分析：循环遍历数组，当数组中的值等于val时，将该元素和从尾部算起不等于val的元素互换；当头部元素和尾部元素相交，跳出该次循环。


## 代码
```
class Solution {
    public int removeElement(int[] nums, int val) {
	//记录尾部开始计算，等于val的数组元素个数
        int hasChange = 0;
        int len = nums.length;
        
        for(int i=0;i<len;i++){
		//剩下的可能需要处理的元素
            int chanLen = len-hasChange-1;
            if(i>chanLen) break;
            
            if(nums[i]==val){
                for(int j=chanLen;j>=0;j--){
				//num[j]在num[i]前面，结束循环
                    if(i>j) break;
                    
                    if(nums[j]!=val){
                        int temp = nums[i];
                        nums[i] = nums[j];
                        nums[j] = temp;
                        hasChange++;
                        break;
                    }
                    
                    hasChange++;
                }
            }
            
        }
        return len-hasChange;
    }
}
```