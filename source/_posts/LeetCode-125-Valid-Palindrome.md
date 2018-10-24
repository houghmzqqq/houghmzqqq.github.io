---
title: 'LeetCode:125.Valid Palindrome'
date: 2017-08-14 13:44:45
tags:
	- LeetCode-频度5
	- 算法
	- Easy
---


## 思路
原题：Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring cases.
For example,
>	"A man, a plan, a canal: Panama" is a palindrome.
>	"race a car" is not a palindrome.

<!-- more -->
大意：判断给定的一个字符串是不是有效的回文（忽略空格和除字母和数字意外的符号）

分析：从头部和尾部同时开始进行比较判断，遇到无效字符就跳过。

## 代码
```
public class Solution {
    public boolean isPalindrome(String s) {
        
        char[] chars = s.toCharArray();
        
        int t = chars.length - 1;
        for(int h=0;h<chars.length;h++){
            if(!Character.isDigit(chars[h]) && !Character.isLetter(chars[h])) continue;
            else if(!Character.isDigit(chars[t]) && !Character.isLetter(chars[t])){
                t--;
                h--;
                continue;
            }

            if(h>=t) return true;
            //转换大小写
            if(Character.isLetter(chars[h]) || Character.isLetter(chars[t])){
                chars[h] = Character.toLowerCase(chars[h]);
                chars[t] = Character.toLowerCase(chars[t]);
            }
            if(chars[h]!=chars[t]) return false;
            
            t--;
        }
        return true;
    }
}
```