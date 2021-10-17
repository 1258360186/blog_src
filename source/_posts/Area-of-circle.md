---
title: 圆的面积
comments: true
mathjax: true
tags:
  - C++
  - 基础语法
categories:
  - 语法基础
date: 2021-10-17 17:54:52
---
#### 音乐小港
{% meting "1824477808" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  圆的面积

### 题目

计算圆的面积的公式定义为 $A=πR^2$。

请利用这个公式计算所给圆的面积。

$π$ 的取值为 $3.14159$。

### 输入格式

输入包含一个浮点数，为圆的半径 $R$。

### 输出格式

输出格式为 `A=X`，其中 $X$ 为圆的面积，用浮点数表示，保留四位小数。

### 数据范围

$0<R<10000.00$

### 输入样例

```
2.00
```

### 输出样例

```
A=12.5664
```

### AC代码

```c++
#include<cstdio>

int main()
{
    double pi = 3.14159, r;
    scanf("%lf",&r);
    printf("A=%.4lf",pi*r*r);
}
```

### 解题思路