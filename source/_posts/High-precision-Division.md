---
title: 高精度除法
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-08 19:58:44
---
#### 音乐小港
{% meting "1426777560" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 高精度除法

### 题目

给定两个非负整数A，B，请你计算 A / B的商和余数。

### 输入格式

共两行，第一行包含整数A，第二行包含整数B。

### 输出格式

共两行，第一行输出所求的商，第二行输出所求余数。

### 数据范围

$1≤A的长度≤100000$,
$1≤B≤10000$
$B 一定不为0$

### 输入样例

```
7
2
```

### 输出样例

```
3
1
```

### AC代码

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

vector<int> div(auto &A,int &b,int &r)
{
    vector<int> C;
    r=0;
    for(int i=A.size()-1;i>=0;i--)
    {
        r=r*10+A[i];
        C.push_back(r/b);
        r%=b;
    }
    reverse(C.begin(),C.end());
    while(C.size()>1&&C.back()==0) C.pop_back();
    return C;
}

int main()
{
    string a;
    int b;
    vector<int> A;
    cin >> a>> b;
    for(int i=a.size()-1;i>=0;i--) A.push_back(a[i]-'0');
    int r=0;
    auto C=div(A,b,r);
    for(int i=C.size()-1;i>=0;i--) cout << C[i];
    puts("");
    cout << r << endl;
    return 0;
}
```

### 解题思路

>**高精度除法**