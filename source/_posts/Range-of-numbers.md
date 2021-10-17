---
title: 数的范围
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-08 19:50:13
---
#### 音乐小港
{% meting "29922545" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 数的范围

### 题目

给定一个按照升序排列的长度为n的整数数组，以及 q 个查询。

对于每个查询，返回一个元素k的起始位置和终止位置（位置从0开始计数）。

如果数组中不存在该元素，则返回“-1 -1”。

### 输入格式

第一行包含整数n和q，表示数组长度和询问个数。

第二行包含n个整数（均在1~10000范围内），表示完整数组。

接下来q行，每行包含一个整数k，表示一个询问元素。

### 输出格式

共q行，每行包含两个整数，表示所求元素的起始位置和终止位置。

如果数组中不存在该元素，则返回“-1 -1”。

### 数据范围

$1≤n≤100000$
$1≤q≤10000$
$1≤k≤10000$

### 输入样例

```
6 3
1 2 2 3 3 4
3
4
5
```

### 输出样例

```
3 4
5 5
-1 -1
```

### AC代码

```c++
#include <cstdio>
using namespace std;

const int N=1e5+10;
int a[N];

int main()
{
    int n,q,k;
    scanf("%d%d",&n,&q);
    for(int i=0;i<n;i++) scanf("%d",&a[i]);
    while(q--)
    {
        scanf("%d",&k);
        int l=0,r=n-1;
        while(l<r)
        {
            int mid=(l+r)>>1;
            if(a[mid]<k) l=mid+1;
            else r=mid;
        }
        if(a[l]!=k)
        {
            printf("-1 -1\n");
        }
        else
        {
            printf("%d ",l);
            int l = 0, r = n - 1;
            while(l<r)
            {
                int mid=(l+r+1)>>1;
                if(a[mid] > k) r=mid-1;
                else l=mid;
            }
            printf("%d \n",r);
        }
    }
    return 0;
}
```

### 解题思路

>**整数二分**
