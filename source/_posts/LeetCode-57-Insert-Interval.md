---
title: 'LeetCode:57.Insert Interval'
date: 2017-08-07 21:09:56
tags:
	- LeetCode-频度5
	- 算法
	- Hard
---

## 思路
原题：Given a set of non-overlapping intervals, insert a new interval into the intervals (merge if necessary).

You may assume that the intervals were initially sorted according to their start times.

<!-- more -->
大意：给一个不重合的时间间隔的集合，插入一个新的间隔，进行排列并合并重合的间隔。该集合按照start升序排列。

分析：可以先将插入的数据按照start排列进入集合中，然后再逐个合并重合的间隔。在这里，合并时分为向上合并和向下合并。向上合并只有可能和插入后的上一个元素合并。

## 代码
```
/**
 * Definition for an interval.
 * public class Interval {
 *     int start;
 *     int end;
 *     Interval() { start = 0; end = 0; }
 *     Interval(int s, int e) { start = s; end = e; }
 * }
 */
public class Solution {
    public List<Interval> insert(List<Interval> intervals, Interval newInterval) {
        
        Interval inta, intb;
        int index = 0;		//记录插入元素的位置
        
        //将插入的间隔按start插入集合中
        intervals.add(newInterval);
        for(int i=intervals.size()-1;i>0;i--){
             if(intervals.get(i).start<intervals.get(i-1).start){
                 Interval temp = intervals.get(i);
                 intervals.set(i,intervals.get(i-1));
                 intervals.set(i-1,temp);
             }else{
                 index = i;
                 break;
             }  
        }
        
        //判断是否和上一个元素合并
        if(index != 0){
            inta = intervals.get(index-1);
            intb = intervals.get(index);
            if(inta.start<=intb.start && inta.end>=intb.start){
                intervals.get(index-1).end = Math.max(inta.end,intb.end);
                intervals.remove(index);
                index--;
            }
        }
        
        //循环判断是否和后面的元素合并
        while(true){
            if(index+1>=intervals.size()){
                break;
            }
            
            inta = intervals.get(index);
            intb = intervals.get(index+1);
            if(inta.end>=intb.start){
                intervals.get(index).end = Math.max(inta.end,intb.end);
                intervals.remove(index+1);
            }else{
                break;
            }
        }
        
        return intervals;
    }
}
```
