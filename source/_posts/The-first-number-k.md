---
title: 第K个数
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-01 19:35:20
---
#### 音乐小港
{% meting "1426572175" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 第k个数

### 题目

给定一个长度为n的整数数列，以及一个整数k，请用快速选择算法求出数列从小到大排序后的第k个数。

### 输入格式

第一行包含两个整数 n 和 k。

第二行包含 n 个整数（所有整数均在$1$ ~ $10^9$范围内），表示整数数列。

### 输出格式

输出一个整数，表示数列的第k小数。

### 数据范围

$1≤n≤100000$,
$1≤k≤n$

### 输入样例

```
5 3
2 4 1 5 3
```

### 输出样例

```
3
```

### AC代码

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int n,k;
int q[100010];

int q_sort(int q[],int x,int y,int k)
{
    if(x>=y) return q[x];
    int l=x-1,r=y+1,m=q[x+y>>1];
    while(l<r)
    {
        do l++;while(q[l]<m);
        do r--;while(q[r]>m);
        if(l<r) swap(q[l],q[r]);
    }
    if(r-x+1>=k) q_sort(q,x,r,k);
    else q_sort(q,r+1,y,k-(r-x+1));
}

int main()
{
    cin >> n >> k;
    for(int i=0;i<n;i++) cin >> q[i];
    cout << q_sort(q,0,n-1,k) << endl;
    return 0;
}
```

### 解题思路

>**快排模板变形**