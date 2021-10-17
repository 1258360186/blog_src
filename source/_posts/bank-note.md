---
title: 钞票
comments: true
mathjax: true
tags:
  - C++
  - 基础语法
categories:
  - 语法基础
date: 2021-10-17 18:02:26
---
#### 音乐小港
{% meting "1397319874" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  钞票

### 题目

在这个问题中，你需要读取一个整数值并将其分解为多张钞票的和，每种面值的钞票可以使用多张，并要求所用的钞票数量尽可能少。

请你输出读取值和钞票清单。

钞票的可能面值有 $100,50,20,10,5,2,1$。

### 输入格式

输入一个整数 $N$。

### 输出格式

参照输出样例，输出读取数值以及每种面值的钞票的需求数量。

### 数据范围

$0<N<1000000$

### 输入样例

```
576
```

### 输出样例

```
576
5 nota(s) de R$ 100,00
1 nota(s) de R$ 50,00
1 nota(s) de R$ 20,00
0 nota(s) de R$ 10,00
1 nota(s) de R$ 5,00
0 nota(s) de R$ 2,00
1 nota(s) de R$ 1,00
```

### AC代码

```c++
#include<cstdio>

int main()
{
    int men;
    scanf("%d",&men);
    printf("%d\n",men);
    printf("%d nota(s) de R$ 100,00\n",men/100);
    men-=men/100*100;
    printf("%d nota(s) de R$ 50,00\n",men/50);
    men-=men/50*50;
    printf("%d nota(s) de R$ 20,00\n",men/20);
    men-=men/20*20;
    printf("%d nota(s) de R$ 10,00\n",men/10);
    men-=men/10*10;
    printf("%d nota(s) de R$ 5,00\n",men/5);
    men-=men/5*5;
    printf("%d nota(s) de R$ 2,00\n",men/2);
    men-=men/2*2;
    printf("%d nota(s) de R$ 1,00",men);
    return 0;
}
```

### 解题思路