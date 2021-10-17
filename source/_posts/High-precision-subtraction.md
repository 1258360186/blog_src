---
title: 高精度减法
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-08 19:56:15
---
#### 音乐小港
{% meting "465675149" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 高精度减法

### 题目

给定两个正整数，计算它们的差，计算结果可能为负数。

### 输入格式

共两行，每行包含一个整数。

### 输出格式

共一行，包含所求的差。

### 数据范围

$1≤整数长度≤100000$

### 输入样例

```
32
11
```

### 输出样例

```
21
```

### AC代码

```c++
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>

using namespace std;

bool cmp(auto &a,auto &b)
{
    if(a.size()!=b.size()) return a.size()>b.size();
    else for(int i=a.size()-1;i>=0;i--) if(a[i]!=b[i]) return a[i]>b[i];
    return true;
}

vector<int> sub(auto &a,auto &b)
{
    vector <int> c;
    int t=0;
    for(int i=0;i<a.size();i++)
    {
        t=a[i]-t;
        if(i<b.size()) t-=b[i];
        c.push_back((t+10)%10);
        if(t<0) t=1;
        else t=0;
    }
    while(c.size()>1&&c.back()==0) c.pop_back();
    return c;
}

int main()
{
    string m,n;
    vector<int> a,b;
    cin >> m >> n;
    for(int i=m.size()-1;i>=0;i--) a.push_back(m[i]-'0');
    for(int i=n.size()-1;i>=0;i--) b.push_back(n[i]-'0');
    if(cmp(a,b)) 
    {
        auto c=sub(a,b);
        for(int i=c.size()-1;i>=0;i--) cout << c[i];
    }else 
    {
        auto c=sub(b,a);
        cout << '-';
        for(int i=c.size()-1;i>=0;i--) cout << c[i];
    }
    return 0;
}
```

### 解题思路

>**高精度减法**

