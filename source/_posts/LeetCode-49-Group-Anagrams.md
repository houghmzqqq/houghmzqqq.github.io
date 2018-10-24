---
title: 'LeetCode:49.Group Anagrams'
date: 2017-09-01 23:10:31
tags:
	- LeetCode-频度5
	- 算法
	- Medium
	- 递归
	- 排列组合

---


## 思路
原题：Given an array of strings, group anagrams together.

大意：给定一个String数组，将拥有相同字母的字符串归为一类。

<!-- more -->
分析：将每个单词按照a~z顺序排序，将排序后相同的字符串放入同一个集合中。

## 代码
```
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        
        List<List<String>> res = new ArrayList();
		//存放排序后的字符串，和原字符串
        Map<String,List<String>> map = new HashMap();
        
        for(String str : strs){
			//对每一个字符串进行重组
            char[] c = str.toCharArray();
            Arrays.sort(c);
            String tempStr = String.valueOf(c);

			//重组后，若未出现过，创建新的集合存放原字符串；
			//若之前出现过，取出与该字符串相对应的集合，将原字符串放入。
            if(map.get(tempStr)==null){
                List<String> l1 = new ArrayList();
                l1.add(str);
                map.put(tempStr,l1);
            }else{
                map.get(tempStr).add(str);
            }
            
        }
        
        for(Map.Entry<String,List<String>> entry : map.entrySet()){
            res.add(entry.getValue());
        }
        return res;
    }
}
```