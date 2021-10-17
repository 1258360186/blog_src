---
title: 工资
comments: true
mathjax: true
tags:
  - C++
  - 基础语法
categories:
  - 语法基础
date: 2021-10-17 17:57:24
---
#### 音乐小港
{% meting "1824477808" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##   工资

### 题目

请编写一个程序，可以读取一名员工的员工编号，本月工作总时长（小时）以及时薪，并输出他的工资条，工资条中包括员工编号和员工月收入。

### 输入格式

输入包含两个整数和一个浮点数，分别代表员工编号，工作时长以及时薪。

每个数占一行。

### 输出格式

输出共两行，第一行格式为 `NUMBER = X`，其中 $X$ 为员工编号。

第二行格式为 `SALARY = U$ Y`，其中 $Y$ 为该员工月收入，保留两位小数。

### 数据范围

$1≤员工编号≤100$,
$1≤总工作时长≤200$,
$1≤时薪≤50$

### 输入样例

```
25
100
5.50
```

### 输出样例

```
NUMBER = 25
SALARY = U$ 550.00
```

### AC代码

```c++
#include<cstdio>

int main()
{
    int num,tim;
    float men;
    scanf("%d%d%f",&num,&tim,&men);
    printf("NUMBER = %d\nSALARY = U$ %.2f",num,tim*men);
    return 0;
}
```

### 解题思路