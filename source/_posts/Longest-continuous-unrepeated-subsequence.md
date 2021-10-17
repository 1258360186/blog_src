---
title: 最长连续不重复子序列
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-13 17:02:24
---
#### 音乐小港
{% meting "1814971662" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  最长连续不重复子序列

### 题目

给定一个长度为n的整数序列，请找出最长的不包含重复的数的连续区间，输出它的长度。

### 输入格式

第一行包含整数n。

第二行包含n个整数（均在0~100000范围内），表示整数序列。

### 输出格式

共一行，包含一个整数，表示最长的不包含重复的数的连续区间的长度。

### 数据范围

$1≤n≤100000$

### 输入样例

```
5
1 2 2 3 5
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

int n;
int q[100010],st[100010];

int main()
{
    cin >> n;
    for(int i=0;i<n;i++) cin >> q[i];
    int res=0;
    for(int i=0,j=0;i<n;i++)
    {
        st[q[i]]++;
        while(j<i&&st[q[i]]>1) st[q[j++]]--;
        res=max(res,i-j+1);
    }
    cout << res << endl;
    return 0;
}
```

### 解题思路

>**双指针算法**

> $st[]$ 数组用来存储指针范围内各元素出现次数；
>
> $j$为前指针，$i$为后指针；
>
> 若后指针移动后，出现重复的数，即将$j$变为$i$同时清空之前已存储的各元素个数；