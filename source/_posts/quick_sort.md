---
title: 快速排序
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-01 19:20:35
---
#### 音乐小港
{% meting "1341556036" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 快速排序

### 题目

给定你一个长度为n的整数数列。

请你使用快速排序对这个数列按照从小到大进行排序。

并将排好序的数列按顺序输出。

### 输入格式

输入共两行，第一行包含整数 n。

第二行包含 n 个整数（所有整数均在$1$ ~ $10^9$范围内），表示整个数列。

### 输出格式

输出共一行，包含 n 个整数，表示排好序的数列。

### 数据范围

$1≤N≤100000$

### 输入样例

```
5
3 1 2 4 5
```

### 输出样例

```
1 2 3 4 5
```

### AC代码

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int n;
int q[100010];

void q_sort(int q[],int x,int y)
{
    if(x>=y) return;
    int l=x-1,r=y+1,k=q[x+y>>1];
    while(l<r)
    {
        do l++;while(q[l]<k);
        do r--;while(q[r]>k);
        if(l<r) swap(q[l],q[r]);
    }
    q_sort(q,x,r);
    q_sort(q,r+1,y);
}

int main()
{
    cin >> n;
    for(int i=0;i<n;i++) cin >> q[i];
    q_sort(q,0,n-1);
    for(int i=0;i<n;i++) cout << q[i] << ' ';
    puts("");
    return 0;
}
```

### 解题思路

>**快排模板**