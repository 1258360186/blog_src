---
title: 二进制中1的个数
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-13 17:40:59
---
#### 音乐小港
{% meting "433681275" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  二进制中1的个数

### 题目

给定一个长度为n的数列，请你求出数列中每个数的二进制表示中1的个数。

### 输入格式

第一行包含整数n。

第二行包含n个整数，表示整个数列。

### 输出格式

共一行，包含n个整数，其中的第 i 个数表示数列中的第 i 个数的二进制表示中1的个数。

### 数据范围

$1≤n≤100000$,
$0≤数列中元素的值≤10^9$

### 输入样例

```
5
1 2 3 4 5
```

### 输出样例

```
1 1 2 1 2
```

### AC代码

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int n;

int main()
{
    cin >> n;
    while(n--)
    {
        int res=0;
        int num=0;
        cin >> num;
        while(num) res++,num-=num&-num;
        //(num&-num)lowbit操作返回最后一位1后的所有位
        cout << res << ' ';
    }
    return 0;
}
```

### 解题思路

>**位运算**