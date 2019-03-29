---
title: 'LeetCode:67.Add Binary'
date: 2017-09-02 15:52:37
tags:
	- LeetCode-频度5
	- 算法
	- Hard

---


## 思路
原题：Given two binary strings, return their sum (also a binary string).

大意：给定一个二进制字符串，输出它们的和，结果也是一个二进制。

<!-- more -->
分析：这道题中，将两个字符串拆分成字符数组，逐位进行相加即可，要注意进位标志。



## 代码
```
class Solution {
    public String addBinary(String a, String b) {
        int c = 0;
        char[] ac = a.toCharArray();
        char[] bc = b.toCharArray();
        int aLen = ac.length-1;
        int bLen = bc.length-1;
		//存放结果，由于从低位开始运算，输出时需要将结果倒转
        StringBuffer buffer = new StringBuffer();
        
		//进行二进制加法运算
        while(aLen>=0 && bLen>=0){
            if((ac[aLen]=='1' && bc[bLen]=='0') || (ac[aLen]=='0' && bc[bLen]=='1')){
                if(c==1) {
                    buffer.append("0");
                    c = 1;
                }else{
                    buffer.append("1");
                }
            }
            else if((ac[aLen]=='0' && bc[bLen]=='0')){
                if(c==1){
                    buffer.append("1");
                    c = 0;
                }else{
                    buffer.append("0");
                }
            }
            else if(ac[aLen]=='1' && bc[bLen]=='1'){
                if(c==1){
                    buffer.append("1");
                }else{
                    buffer.append("0");
                }
                c = 1;
            }
            aLen--;
            bLen--;
        }
        
		//其中一个或两个数都相加完，处理剩余的进位标志
        if(aLen<0 || bLen<0){
            int max = Math.max(aLen,bLen);
            char[] temp;
            if(bLen<0) temp = ac;
            else temp = bc;
            
            for(int i=max;i>=0;i--){
                if(c==1 && temp[i]=='0'){
                    buffer.append("1");
                    c = 0;
                }
                else if(c==1 && temp[i]=='1'){
                    buffer.append("0");
                    c = 1;
                }
                else if(c==0){
                    buffer.append(temp[i]);
                }
                
            }
            
        }
        
        if(c==1){
            buffer.append("1");
        }
        
        return buffer.reverse().toString();
    }
}
```