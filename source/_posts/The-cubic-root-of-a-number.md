---
title: 数的三次方根
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-08 19:51:33
---
#### 音乐小港
{% meting "411315504" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 数的三次方根

### 题目

给定一个浮点数n，求它的三次方根。

### 输入格式

共一行，包含一个浮点数n。

### 输出格式

共一行，包含一个浮点数n。

### 数据范围

$−10000≤n≤10000$

### 输入样例

```
1000.00
```

### 输出样例

```
10.000000
```

### AC代码

```c++
#include <cstdio>
using namespace std;


int main()
{
    double n;
    scanf("%lf",&n);
    double l=-100,r=100;
    while(r-l>1e-8)
    {
        double mid=(l+r)/2;
        if(mid*mid*mid<n) l=mid;
        else r=mid;
    }
    printf("%.6lf",l);
    return 0;
}
```

### 解题思路

>**实数二分**