---
title: BigDecimal控制精度及BigInteger解决整数溢出
date: 2017-10-13 22:15:57
tags:
	- Java

---

## 1. BigDecimal

### 1.1 进行精确计算
```
public void test(){
        System.out.println(0.06 + 0.01);
        System.out.println(1.0 - 0.42);
    }
```

执行以上代码的结果是：

	0.06999999999999999
	0.5800000000000001

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;原因在于我们的计算机是二进制的。浮点数没有办法是用二进制进行精确表示。我们的CPU表示浮点数由两个部分组成：指数和尾数，这样的表示方法一般都会失去一定的精确度，有些浮点数运算也会产生一定的误差。如：2.4的二进制表示并非就是精确的2.4。反而最为接近的二进制表示是 2.3999999999999999。浮点数的值实际上是由一个特定的数学公式计算得到的。

<!-- more -->
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以使用BigDecimal进行精确运算

```
public static double add(double value1,double value2){
        BigDecimal b1 = new BigDecimal(String.valueOf(value1));
        BigDecimal b2 = new BigDecimal(String.valueOf(value2));
        return b1.add(b2).doubleValue();
    }
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里需要使用new BigDecimal(String str)构造方法实例化BigDecimal，然后通过内置的add()方法进行精确加法运算(相应的subtract()、multiply()、divide()方法进行其他运算)。__[因为使用new BigDecimal(Double d) 时，得到的BigDecimal中的long值并不是精确的，使用new BigDecimal(String str)才能达到精确运算的目的]__

### 1.2 控制精度

有时会遇到保留小数点后两位，这样的要求精度的需求。可以使用BigDecimal控制。

```
BigDecimal b1 = new BigDecimal(0.33333);
BigDecimal b2 = b1.setScale(2,BigDecimal.ROUND_HALF_UP);
```
其中`b1.setScale(2,BigDecimal.ROUND_HALF_UP)`表示设置b1的精度为2并且采用四舍五入的策略

## 2. BigInteger

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;java中常规的进行整数运算的类有两个，Integer和Long，其中Long的表示范围比Integer要大，Long能够表示的最大值为2^63-1 = 9223372036854775807，当需要进行运算的数超过这个值时会发生溢出。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们可以使用BigInteger进行大整数的运算。

```
BigInteger bi1 = new BigInteger(String.valueOf(10));
BigInteger bi2 = bi1.multiply(new BigInteger("100000000000000000000000000000000000000000000"));
```


