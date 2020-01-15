---
layout: post
title: "BigDecimal学习"
categories: [Java]
description:
keywords:
---

* content
{:toc}

## 练习

```java
public static void main(String[] args){
    // ROUND_UP 强制进位, 向远离0的方向   2.39, 2.34  ->  2.4     -2.39, -2.34  ->  -2.4
    // ROUND_DOWN 强制舍去, 向零方向舍入  2.39, 2.34  ->  2.3     -2.39, -2.34  ->  2.3
    // ROUND_HALF_UP 四舍五入   2.35  ->  2.4   2.34  ->  2.3   -2.35  >  -2.4  -2.34  ->  -2.3
    // ROUND_HALF_DOWN 四舍五入(但是5时也会舍)  2.36  ->  2.4   2.35  ->  2.3   -2.36  >  -2.4  -2.35  ->  -2.3
    // ROUND_CEILING 强制进位, 向正无穷方向   2.39, 2.34 -> 2.4   -2.39, -2.34 -> -2.3
    // ROUND_FLOOR 强制进位, 向负无穷方向   2.39, 2.34 -> 2.3   -2.39, -2.34 -> -2.4
    // ROUND_HALF_EVEN 向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，如果保留位数是偶数，使用ROUND_HALF_UP ，如果是奇数，使用ROUND_HALF_DOWN
    // ROUND_UNNECESSARY 不需要进位
    System.out.println(new BigDecimal("1.55").setScale(1, BigDecimal.ROUND_HALF_EVEN));
}    
```
