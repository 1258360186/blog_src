---
title: 数组元素的目标和
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-13 17:08:39
---
#### 音乐小港
{% meting "1501365659" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  数组元素的目标和

### 题目

给定两个升序排序的有序数组A和B，以及一个目标值x。数组下标从0开始。
请你求出满足A[i] + B[j] = x的数对(i, j)。

数据保证有唯一解。

### 输入格式

第一行包含三个整数n，m，x，分别表示A的长度，B的长度以及目标值x。

第二行包含n个整数，表示数组A。

第三行包含m个整数，表示数组B。

### 输出格式

共一行，包含两个整数 i 和 j。

### 数据范围

数组长度不超过100000。
同一数组内元素各不相同。
$1≤数组元素≤10^9$

### 输入样例

```
4 5 6
1 2 4 7
3 4 6 8 9
```

### 输出样例

```
1 1
```

### AC代码

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int n,m,x;
int A[100010],B[100010];

int main()
{
    cin >> n >> m >> x;
    for(int i=0;i<n;i++) cin >> A[i];
    for(int i=0;i<m;i++) cin >> B[i];
    for(int i=0,j=m-1;i<n;i++)
    {
        int temp=x-A[i];
        while(j>=0&&B[j]>temp) j--;
        if(B[j]==temp)
        {
            cout << i <<' '<< j << endl;
            break;
        }
    }
    return 0;
}
```

### 解题思路

>**双指针算法**

> 因为两个序列都是升序序列，故若某一序列从大到小遍历，为满足题意，从另一个序列取的值比如会越来越小；
>
> 序列不存在重复元素，下个元素对应的数必然在另一个序列的更前面；