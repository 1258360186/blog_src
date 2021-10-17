---
title: 逆序对的数量
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-02 21:10:36
---
#### 音乐小港
{% meting "1464325108" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 逆序对的数量

### 题目

给定一个长度为n的整数数列，请你计算数列中的逆序对的数量。

逆序对的定义如下：对于数列的第 i 个和第 j 个元素，如果满足 i < j 且 a[i] > a[j]，则其为一个逆序对；否则不是。

### 输入格式

第一行包含整数n，表示数列的长度。

第二行包含 n 个整数，表示整个数列。

### 输出格式

输出一个整数，表示逆序对的个数。

### 数据范围

$1≤n≤100000$

### 输入样例

```
6
2 3 4 5 6 1
```

### 输出样例

```
5
```

### AC代码

```c++
#include <iostream>
using namespace std;
using LL = long long;
int n;
int q[100010],tmp[100010];
LL res;

LL m_sort(int q[],int l,int r)
{
    LL res=0;
    if(l>=r) return res;
    int mid=l+r>>1;
    res=m_sort(q,l,mid)+m_sort(q,mid+1,r);
    int i=l,j=mid+1,k=0;
    while(i<=mid&&j<=r)
    {
        if(q[i]<=q[j]) tmp[k++]=q[i++];
        else tmp[k++]=q[j++],res+=mid-i+1;
    }
    while(i<=mid) tmp[k++]=q[i++];
    while(j<=r) tmp[k++]=q[j++];
    for(int i=l,k=0;i<=r;i++) q[i]=tmp[k++];
    return res;
}

int main()
{
    scanf("%d",&n);
    for(int i=0;i<n;i++) scanf("%d",&q[i]);
    printf("%lld",m_sort(q,0,n-1));
    return 0;
}

```

### 解题思路

>**归并排序模板变形**
>
>所求的逆序对恰好是逆序数量；