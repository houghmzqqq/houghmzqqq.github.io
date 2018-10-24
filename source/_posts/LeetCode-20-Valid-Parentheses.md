---
title: 'LeetCode:20.Valid Parentheses'
date: 2017-07-23 14:34:25
tags:
    - LeetCode-频度5
    - 算法
    - Easy
---


## 思路
原题：Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

The brackets must close in the correct order, "()" and "()[]{}" are all valid but "(]" and "([)]" are not.

大意：给一个只包含'(', ')', '{', '}', '[' , ']'等字符的字符串，判断该字符串中的所有括号是否有效。

<!-- more -->
分析：根据题意，可以判断出有效的括号 like_this"()","()[]","({})"，无效括号like_this"((","]]","([)]","()["。

步骤：
	1、排除明显错误的答案（字符数量为单数时）
	2、遍历所有字符，遇到开括号(如"(","[","{")时，放入list中
	3、遇到闭括号，与list中的最后一个字符匹配


## 代码
```
public class Solution {
    public boolean isValid(String s) {
        char[] paren = s.toCharArray();
        List<Character> openParen = new ArrayList(); //存放开括号
        
        //排除明显错误的答案
        if(paren.length%2 != 0)
        {
            return false;
        }

        for(Character p : paren){
            int type;
            if( p=='(' || p=='[' || p=='{' ){
                openParen.add(p);
            }
            if( p==')' || p==']' || p=='}' ){
                if(openParen.size()==0){
                    return false;
                }
                int lastIndex = openParen.size()-1;
                
                if(!(openParen.get(lastIndex)=='(' && p==')' ||
                  openParen.get(lastIndex)=='[' && p==']' ||
                  openParen.get(lastIndex)=='{' && p=='}')){
                    return false;
                }else{
                    openParen.remove(lastIndex);
                }
            }
        }
        if(openParen.size()!=0){
            return false;
        }
        return true;
    }
}
```