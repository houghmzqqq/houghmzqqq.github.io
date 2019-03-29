---
title: 'LeetCode:79.Word Search'
date: 2017-09-02 16:19:39
tags:
	- LeetCode-频度5
	- 算法
	- Medium
	- 连通图
	- 深度优先搜素DFS

---

## 思路
原题：Given a 2D board and a word, find if the word exists in the grid.

The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.

大意：给定一个2D的字母表和一个单词，从该表中找出该单词。
规则--单词可以由相邻相邻单元格的字母构成，相邻的单元格是水平或垂直相邻的单元格。同一个字母单元不能超过一次使用。

<!-- more -->
分析：对每一个点都进行深度遍历，遍历过程中发生：1.数组超出界限；2.指定位置的单词字母与该点字母不相等；3.给点已经遍历过。要进行剪枝。


## 代码
```
class Solution {
    public boolean exist(char[][] board, String word) {
        char[] cWord = word.toCharArray();
        boolean[][] isUsed = new boolean[board.length][board[0].length];
        
        Queue<Character> queue = new LinkedList();
        
        for(int i=0;i<board.length;i++){
            for(int j=0;j<board[0].length;j++){
                // if(cWord[0]==board[i][j]){
                    if(judge(isUsed,board,cWord,0,i,j)) return true;
                // }
            }
        }
        return false;
    }
    
    public boolean judge(boolean[][] isUsed,char[][] board,char[] cw,int w,int x,int y){
        int bLen = board.length;
        int bHig = board[0].length;
		// 坐标可以向上下左右移动
        int[] bh = {1,0,-1,0};
        int[] bw = {0,1,0,-1};
        
        if(w==cw.length) return true;
		//剪枝
        if(x<0 || y<0 || x>=bLen || y>=bHig || cw[w]!=board[x][y] || isUsed[x][y]==true) return false;
        isUsed[x][y] = true;
        
        for(int i=0;i<4;i++){
            if(judge(isUsed,board,cw,w+1,x+bh[i],y+bw[i])) return true;
        }
        isUsed[x][y] = false;
        return false;
    }
}
```