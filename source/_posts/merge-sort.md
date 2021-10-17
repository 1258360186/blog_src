---
title: 归并排序
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-02 21:08:21
---
#### 音乐小港
{% meting "34766078" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 归并排序

### 题目

给定你一个长度为n的整数数列。

请你使用归并排序对这个数列按照从小到大进行排序。

并将排好序的数列按顺序输出。

### 输入格式

输入共两行，第一行包含整数 n。

第二行包含 n 个整数（所有整数均在$1-10^9$范围内），表示整个数列。

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
int a[100010],b[100010];

void m_sort(int a[],int x,int y)
{
    if(x>=y) return;
    int mid = (x+y)>>1;
    m_sort(a,x,mid);
    m_sort(a,mid+1,y);
    int l=x,r=mid+1,k=0;
    while(l<=mid&&r<=y)
    {
        if(a[l]<a[r]) b[k++]=a[l++];
        else b[k++]=a[r++];
    }
    while(l<=mid) b[k++]=a[l++];
    while(r<=y) b[k++]=a[r++];
    for(int i=x,j=0;i<=y;i++,j++) a[i]=b[j];
}

int main()
{
    cin >> n;
    for(int i=0;i<n;i++) cin >> a[i];
    m_sort(a,0,n-1);
    for(int i=0;i<n;i++) cout << a[i] << ' ';
    return 0;
}
```

### 解题思路

>**归并排序模板**