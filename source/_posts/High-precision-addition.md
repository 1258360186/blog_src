---
title: 高精度加法
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-08 19:55:02
---
#### 音乐小港
{% meting "448741168" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 高精度加法

### 题目

给定两个正整数，计算它们的和。

### 输入格式

共两行，每行包含一个整数。

### 输出格式

共一行，包含所求的和。

### 数据范围

$1≤整数长度≤100000$

### 输入样例

```
12
23
```

### 输出样例

```
35
```

### AC代码

```c++
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>

using namespace std;

vector<int> add(auto &a,auto &b)
{
    if(a.size()<b.size()) return add(b,a);
    vector <int> c;
    int t=0;
    for(int i=0;i<a.size();i++)
    {
        t+=a[i];
        if(i<b.size()) t+=b[i];
        c.push_back(t%10);
        t/=10;
    }
    if(t) c.push_back(t);
    return c;
}

int main()
{
    string m,n;
    vector<int> a,b;
    cin >> m >> n;
    for(int i=m.size()-1;i>=0;i--) a.push_back(m[i]-'0');
    for(int i=n.size()-1;i>=0;i--) b.push_back(n[i]-'0');
    auto c=add(a,b);
    for(int i=c.size()-1;i>=0;i--) cout << c[i];
    return 0;
}
```

### 解题思路

>**高精度加法**