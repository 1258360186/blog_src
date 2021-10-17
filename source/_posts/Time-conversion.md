---
title: 时间转换
comments: true
mathjax: true
tags:
  - C++
  - 基础语法
categories:
  - 语法基础
date: 2021-10-17 18:03:28
---
#### 音乐小港
{% meting "554375621" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  时间转换

### 题目

读取一个整数值，它是工厂中某个事件的持续时间（以秒为单位），请你将其转换为小时：分钟：秒来表示。

### 输入格式

输入一个整数 $N$。

### 输出格式

输出转换后的时间表示，格式为 `hours:minutes:seconds`。

### 数据范围

$0<N<1000000$

### 输入样例

```
556
```

### 输出样例

```
0：9：16
```

### AC代码

```c++
#include<cstdio>

int main()
{
    int tim;
    scanf("%d",&tim);
    printf("%d:%d:%d",tim/3600,tim%3600/60,tim%3600%60);
    return 0;
}
```

### 解题思路