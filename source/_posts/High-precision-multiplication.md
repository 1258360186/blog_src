---
title: 高精度乘法
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-08 19:57:37
---
#### 音乐小港
{% meting "1464563836" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 高精度乘法

### 题目

给定两个正整数A和B，请你计算A * B的值。

### 输入格式

共两行，第一行包含整数A，第二行包含整数B。

### 输出格式

共一行，包含A * B的值。

### 数据范围

$1≤A的长度≤100000$,
$0≤B≤10000$

### 输入样例

```
2
3
```

### 输出样例

```
6
```

### AC代码

```c++
#include <iostream>
#include <vector>
using namespace std;

vector<int> mul(auto &A,int b)
{
    vector<int> C;
    for(int i=0,t=0;i<A.size()||t;i++){
        if(i<A.size()) t+=A[i]*b;
        C.push_back(t%10);
        t/=10;
    }
    while (C.size() > 1 && C.back() == 0) C.pop_back();
    return C;
}

int main()
{
    string a;
    int b;
    vector<int>A;
    cin >> a>> b;
    for(int i=a.size()-1;i>=0;i--) A.push_back(a[i]-'0');
    auto C=mul(A,b);
    for(int i=C.size()-1;i>=0;i--) cout << C[i];
    return 0;
}
```

### 解题思路

>**高精度乘法**

