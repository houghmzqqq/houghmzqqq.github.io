---
title: 'LeetCode:8.String to Integer (atoi)'
date: 2017-07-19 17:20:19
tags:
	- LeetCode-频度5
	- 算法
	- Medium
---


## 思路
原题：Implement atoi to convert a string to an integer.

<!-- more -->
大意：实现字符串转换成int，参考atoi函数

分析：要求实现atoi函数相似的功能，通过查找资料了解到atoi函数的作用——
>	1.如果字符前面有空格，跳过。
>	2.第一个有效字符可以为‘+’或‘-’号。
>	3.转换过程中如果遇到非数字符号或空格，停止转换，输出当前已转换的数字
__需要注意int的溢出问题__


步骤：
	1、过滤前面的空格 
	2、判断正负
	3、转换有效数字
	4、判断是否溢出。为了防止溢出现象，本人使用Long类型来暂时存储数字，由于long也有边界值会溢出，所以添加一个length变量来记录数字的长度

## 代码

```
public class Solution {
  public int myAtoi(String str) {
		Long s = 0l;
		int length = 0;

		char[] strChar = str.toCharArray();

		boolean theFirst = true;
		boolean isJumpSymbol = false;
		boolean flag = false;

		if (str == null || str.equals(""))
			return 0;

		for (Character c : strChar) {
			// 1.不为空
			if (!Character.isWhitespace(c)) {
				// 检测有效字符，有效字符串前面的空格已经跳过，做上标记
				if (isJumpSymbol == false) {
					isJumpSymbol = true;
				}

				// 2.第一个字符为正负符号
				if ((c.equals('-') || c.equals('+')) && theFirst == true) {
					theFirst = false;

					// 如果为负号，做上标记
					if (c.equals('-')) {
						flag = true;
					}
				} else {
					// 3.是否为数字
					if (Character.isDigit(c)) {
						// 记录有效数字的长度
						if (length > 10)
							break;
						length++;

						switch (c) {
						case '0':
							s = s * 10 + 0;
							break;
						case '1':
							s = s * 10 + 1;
							break;
						case '2':
							s = s * 10 + 2;
							break;
						case '3':
							s = s * 10 + 3;
							break;
						case '4':
							s = s * 10 + 4;
							break;
						case '5':
							s = s * 10 + 5;
							break;
						case '6':
							s = s * 10 + 6;
							break;
						case '7':
							s = s * 10 + 7;
							break;
						case '8':
							s = s * 10 + 8;
							break;
						case '9':
							s = s * 10 + 9;
							break;
						}
					} else {
						break;
					}
				}
			} else if (Character.isWhitespace(c) && isJumpSymbol == true) {
				break;
			}
		}

		// 判断是否溢出
		if (flag) {
			if (-s < Integer.MIN_VALUE)
				return Integer.MIN_VALUE;
		} else {
			if (s > Integer.MAX_VALUE)
				return Integer.MAX_VALUE;
		}

		return (int) (s * (flag ? -1 : 1));
	}
}
```

## 心得
&nbsp;&nbsp;&nbsp;&nbsp;这一题让我理解的测试的重要性，血的教训啊！！！T。T

