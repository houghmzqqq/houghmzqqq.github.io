---
title: 'LeetCode:73.Set Matrix Zeroes'
date: 2017-08-13 12:37:41
tags:
	- LeetCode-频度5
	- 算法
	- Medium
---


## 思路
原题：Given a m x n matrix, if an element is 0, set its entire row and column to 0. Do it in place.

大意：给定一个mXn的矩阵，如果一个元素，将该元素所在的行和列的所有元素设置为0。

<!-- more -->
分析：将第一行和第一列作为工具，先判断第一行和第一列中是否存在0，做上标记；判断其他元素，如果一个元素为0，将该元素所在的第一行和第一列元素标记为0；最后再根据第一行和第一列对所有元素进行转换。

## 代码
```
public class Solution {
    public void setZeroes(int[][] matrix) {
        int rowLength = matrix.length;
        int colLength = matrix[0].length;
        boolean isFirstRowHasZeo = false;
        boolean isFirstColHasZeo = false;
        
        for(int i=0;i<rowLength;i++){
            if(matrix[i][0]==0){
                isFirstColHasZeo = true;
                break;
            }
        }
        for(int i=0;i<colLength;i++){
            if(matrix[0][i]==0){
                isFirstRowHasZeo = true;
                break;
            }
        }
        
        for(int i=1;i<matrix.length;i++){
            for(int j=1;j<matrix[i].length;j++){
                if(matrix[i][j]==0){
                    matrix[i][0] = 0;
                    matrix[0][j] = 0;
                }
            }
        }
        
        for(int i=1;i<rowLength;i++){
            for(int j=1;j<colLength;j++){
                if(matrix[i][0]==0 || matrix[0][j]==0){
                    matrix[i][j] = 0;
                }
            }
        }
        
        if(isFirstRowHasZeo){
            for(int i=0;i<colLength;i++){
                matrix[0][i] = 0;
            }
        }
        if(isFirstColHasZeo){
            for(int i=0;i<rowLength;i++){
                matrix[i][0] = 0;
            }
        }
        
    }
}
```