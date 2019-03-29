---
title: 'LeetCode:56.Merge Intervals'
date: 2017-08-07 15:30:58
tags:
	- LeetCode-频度5
	- 算法
	- Medium
---

！（该答案在LeetCode官网上运行时出现超时错误，需要优化）
## 思路
原题：Given a collection of intervals, merge all overlapping intervals.
> For example,
> 	Given [1,3],[2,6],[8,10],[15,18],
> 	return [1,6],[8,10],[15,18].

<!-- more -->
大意：给一个含有多个时间间隔的集合，合并所有重合的部分。

分析：给出多个时间间隔，它们两两之间存在以下两种种可能性：
	1.两个间隔不相交，如[1,2]和[3,4]
	4.两个间隔相交，如[1,3]和[2,4]

## 代码
```java
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
    public List<Interval> merge(List<Interval> intervals) {
        int count = 0;
        List<Interval> result = new ArrayList();
        List<Integer> temps = new ArrayList();
        
        //先按照start进行升序排序
        for(int i=0;i<intervals.size()-1;i++){
            for(int j=i+1;j<intervals.size();j++){
                if(intervals.get(i).start>intervals.get(j).start){
                    Interval flag = intervals.get(i);
                    intervals.set(i,intervals.get(j));
                    intervals.set(j,flag);
                }
            }
        }
        
        for(int i=0;i<intervals.size();i++){
            Interval inter01 = intervals.get(i);
            
            //判断当前元素之前是否已经被合并
            boolean ifJump = false;
            for(Integer temp : temps){
                if(i==temp){
                    ifJump = true;
                }
            }
            if(ifJump){
                continue;
            }
            
            for(int j=i+1;j<intervals.size();j++){
                Interval inter02 = intervals.get(j);
                
                //第一种情况，两个范围不相交
                if(inter01.end<inter02.start || inter01.start>inter02.end){
                    continue;
                }
                //第二种情况，两个范围存在包含关系
                else if((inter01.start<inter02.start && inter01.end>inter02.end)
                        || (inter01.start>inter02.start && inter01.end<inter02.end)){
                    
                    if(inter01.start>inter02.start && inter01.end<inter02.end){
                        inter01.start = inter02.start;
                        inter01.end = inter02.end;
                    }
                    temps.add(j);
                    
                }
                //第三种情况，有元素相等
                else if(inter01.start==inter02.start || inter01.end==inter02.end){
                    
                    if(inter01.start>inter02.start){
                        inter01.start = inter02.start;
                        inter01.end = inter02.end;
                    }else if(inter01.end<inter02.end){
                        inter01.start = inter02.start;
                        inter01.end = inter02.end;
                    }
                    temps.add(j);
                    
                }
                //第四种情况，两个范围相交
                else if((inter01.start<inter02.start && inter01.end<inter02.end) 
                         || (inter01.start>inter02.start && inter01.end>inter02.end)){
                    
                    if(inter01.start<inter02.start && inter01.end<inter02.end){
                        inter01.end = inter02.end;
                    }else{
                        inter01.start = inter02.start;
                    }
                    temps.add(j);
                }
                
            }
            
            //过滤相等的结果
            if(result.size()==0){
                result.add(inter01);   
            }else if(result.get(count).start==inter01.start && result.get(count).end==inter01.end){
                
            }else{
                result.add(inter01);
                count++;
            }
            
        }
        return result;
    }
}
```


## 优化
分析：先根据start进行排序，然后进行合并，合并条件__a.start <= b.start <= a.end__

## 代码（优）
```java
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
    public List<Interval> merge(List<Interval> intervals) {
        
        //根据interval的start排序
        Collections.sort(intervals, new Comparator<Interval>() {
            @Override
            public int compare(Interval o1, Interval o2) {
                return o1.start - o2.start;
            }
        });
        
        for(int i=0;i<intervals.size()-1;i++){
            Interval inta = intervals.get(i);
            Interval intb = intervals.get(i+1);
            if(inta.start<=intb.start && inta.end>=intb.start){
                intervals.get(i).end = Math.max(inta.end,intb.end);
                intervals.remove(i+1);
                i--;
            }
        }
        
        return intervals;
    }
}
```