---
title: 'LeetCode:127.Word Ladder'
date: 2017-08-14 13:58:46
tags:
	- LeetCode-频度5
	- 算法
	- Medium
---


## 思路
原题：Given two words (beginWord and endWord), and a dictionary's word list, find the length of shortest transformation sequence from beginWord to endWord, such that:
>	1.Only one letter can be changed at a time.
>	2.Each transformed word must exist in the word list. Note that beginWord is not a transformed word.

<!-- more -->
大意：给出两个单词（开始单词和结束单词）和一个字典，查找字典中从开始单词转变为结束单词的最短路径，规则如下：
>	1.一次只能改变一个单词
>	2.每一个转换后的单词必须为字典中的单词。开始单词不算是转换单词（即结束单词算是）。

分析：查找最短路径，首先想到的是数据结构中连通图的广度搜索算法（BFS）。

![t1](t1.PNG)

![t1](t2.PNG)

定义一个栈midList，存放图中的灰色元素，白色元素为列表root，长度存放在Map中

## 代码
```
public class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        
        Queue<String> midList = new LinkedList();
        midList.offer(beginWord);
        Map<String,Integer> map = new HashMap();
        map.put(beginWord,1);
        
        if(!wordList.contains(endWord)) return 0;
        
        if(wordList.contains(beginWord)) wordList.remove(beginWord);
        
        while(!midList.isEmpty()){
            String top = midList.poll();
            StringBuilder builder;
            
            int level = map.get(top);
            for(int i=0;i<top.length();i++){
                for(char c='a';c<='z';c++){
                    builder = new StringBuilder(top);
                    builder.setCharAt(i,c);
                    String tepStr = builder.toString();
                    if(top==tepStr) continue;
                    if(tepStr.equals(endWord)) return level+1;
                    
                    if(wordList.contains(tepStr)){
                        midList.offer(tepStr);
                        wordList.remove(tepStr);
                        map.put(tepStr,level+1);
                    }
                    
                }
                
            }
        }
        return 0;
    }
}
```