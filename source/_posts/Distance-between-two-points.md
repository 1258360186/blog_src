---
title: 两点间的距离
comments: true
mathjax: true
tags:
  - C++
  - 基础语法
categories:
  - 语法基础
date: 2021-10-17 18:01:07
---
#### 音乐小港
{% meting "1378847146" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##   两点间的距离

### 题目

给定两个点 $P_1$ 和 $P_2$，其中 $P_1$ 的坐标为 $(x_1,y_1)$，$P_2$ 的坐标为 $(x_2,y_2)$，请你计算两点间的距离是多少。

​    											$$distance=\sqrt{(x_2−x_1)^2+(y_2−y_1)^2}$$

### 输入格式

输入共两行，每行包含两个双精度浮点数 $x_i,y_i$，表示其中一个点的坐标。

输入数值均保留一位小数。

### 输出格式

输出你的结果，保留四位小数。

### 数据范围

$−{10}^9≤x_i,y_i≤{10}^9$

### 输入样例

```
1.0 7.0
5.0 9.0
```

### 输出样例

```
4.4721
```

### AC代码

```c++
#include<cstdio>
#include<cmath>

int main()
{
    double a,b,c,d;
    scanf("%lf%lf\n%lf%lf",&a,&b,&c,&d);
    printf("%.4lf",sqrt((a-c)*(a-c)+(b-d)*(b-d)));
    return 0;
}
```

### 解题思路