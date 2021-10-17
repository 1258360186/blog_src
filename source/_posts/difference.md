---
title: 差
comments: true
mathjax: true
tags:
  - C++
  - 基础语法
categories:
  - 语法基础
date: 2021-10-17 17:53:08
---
#### 音乐小港
{% meting "1476113773" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 差

### 题目

读取四个整数 $A,B,C,D$，并计算 $(A×B−C×D)$ 的值。

### 输入格式

输入共四行，第一行包含整数 $A$，第二行包含整数 $B$，第三行包含整数 $C$，第四行包含整数 $D$。

### 输出格式

输出格式为 `DIFERENCA = X`，其中 $X$ 为 $(A×B−C×D)$ 的结果。

### 数据范围

$−10000≤A,B,C,D≤10000$

### 输入样例

```
5
6
7
8
```

### 输出样例

```
DIFERENCA = -26
```

### AC代码

```c++
#include<iostream>
#include<cstdio>
using namespace std;
int main()
{
    int a,b,c,d;
    scanf("%d%d%d%d",&a,&b,&c,&d);
    printf("DIFERENCA = %d",a*b-c*d);
    return 0;
}
```

### 解题思路

>****
