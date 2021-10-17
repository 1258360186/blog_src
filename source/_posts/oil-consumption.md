---
title: 油耗
comments: true
mathjax: true
tags:
  - C++
  - 基础语法
categories:
  - 语法基础
date: 2021-10-17 17:58:32
---
#### 音乐小港
{% meting "1475476091" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  油耗

### 题目

给定一个汽车行驶的总路程（$km$）和消耗的油量（$l$），请你求出汽车每消耗 $1$ 升汽油可行驶多少公里路程。

### 输入格式

输入共两行，第一行包含整数 $X$，表示行驶总路程。

第二行包含保留一位小数的浮点数 $Y$，表示消耗的油量。

### 输出格式

输出格式为 `M km/l`，其中 MM 为计算结果，保留三位小数。

### 数据范围

$1≤X,Y≤10^9$

### 输入样例

```
500
35.0
```

### 输出样例

```
14.286 km/l
```

### AC代码

```c++
#include<cstdio>

int main()
{
    int km;
    float l;
    scanf("%d%f",&km,&l);
    printf("%.3f km/l",km/l);
    return 0;
}
```

### 解题思路