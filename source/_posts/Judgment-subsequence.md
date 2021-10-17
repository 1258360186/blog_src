---
title: 判断子序列
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-13 17:19:14
---
#### 音乐小港
{% meting "548823743" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  判断子序列

### 题目

给定一个长度为 n 的整数序列 $a_1,a_2,…,a_n$ 以及一个长度为 m 的整数序列 $b_1,b_2,…,b_m$。

请你判断 a 序列是否为 b 序列的子序列。

子序列指序列的一部分项按原有次序排列而得的序列，例如序列 ${a_1,a_3,a_5}$ 是序列 ${a_1,a_2,a_3,a_4,a_5}$ 的一个子序列。

### 输入格式

第一行包含两个整数 n,m。

第二行包含 n 个整数，表示 $a_1,a_2,…,a_n$。

第三行包含 m 个整数，表示 $b_1,b_2,…,b_m$。

### 输出格式

如果 a 序列是 b 序列的子序列，输出一行 Yes。

否则，输出 No。

### 数据范围

$1≤n≤m≤10^5$,  
$−10^9≤a_i,b_i≤10^9$

### 输入样例

```
3 5
1 3 5
1 2 3 4 5
```

### 输出样例：

```
Yes
```

### AC代码
```C++
#include <iostream>

using namespace std;

int n,m;
int a[100010],b[100010];

bool check(int a[],int b[])
{
    int i=0,j=0;
    while(i<n&&j<m)
    {
        if(a[i]==b[j]) i++,j++;
        else j++;
    }
    return i==n;
}

int main()
{
    cin >> n >>m;
    for(int i=0;i<n;i++) cin >> a[i];
    for(int i=0;i<m;i++) cin >> b[i];
    bool res=check(a,b);
    if(res) puts("Yes");
    else puts("No");
    return 0;
}
```

### 解题思路

>**双指针算法**

> 用两个指针分别指向两个序列，若两个序列对应相同则都向后移动，不匹配则移动原序列。
>
> 遍历整个原序列后若存在个数等于子序列个数，代表子序列是原序列的子序列。