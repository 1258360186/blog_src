---
title: 区间和
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-13 17:44:09
---
#### 音乐小港
{% meting "1379486136" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  区间和

### 题目

假定有一个无限长的数轴，数轴上每个坐标上的数都是0。

现在，我们首先进行 n 次操作，每次操作将某一位置x上的数加c。

接下来，进行 m 次询问，每个询问包含两个整数l和r，你需要求出在区间[l, r]之间的所有数的和。

### 输入格式

第一行包含两个整数n和m。

接下来 n 行，每行包含两个整数x和c。

再接下里 m 行，每行包含两个整数l和r。

### 输出格式

共m行，每行输出一个询问中所求的区间内数字和。

### 数据范围

$−10^9≤x≤10^9$,
$1≤n,m≤10^5$,
$−10^9≤l≤r≤10^9$
$−10000≤c≤10000$

### 输入样例

```
3 3
1 2
3 6
7 5
1 3
4 6
7 8
```

### 输出样例

```
8
0
5
```

### AC代码

```c++
#include <iostream>
#include <algorithm>
#include <vector>

#define x first
#define y second

using namespace std;

typedef pair<int,int> PII;
int n,m;
int num[300010],s[300010];
vector<int> all;
vector<PII> query,add;

int find(int x)
{
    int l=0,r=all.size()-1;
    while(l<r)
    {
        int mid=(l+r)>>1;
        if(all[mid]<x) l=mid+1;
        else r=mid;
    }
    return l+1;
}

int main()
{
    cin >> n >> m;
    while(n--)
    {
        int x,c;
        cin >> x >> c;
        all.push_back(x);
        add.push_back({x,c});
    }
    while(m--)
    {
        int l,r;
        cin >> l >>r;
        all.push_back(l);
        all.push_back(r);
        query.push_back({l,r});
    }
    sort(all.begin(),all.end());
    all.erase(unique(all.begin(),all.end()),all.end());
    for(auto item:add)
    {
        int idx=find(item.x);
        num[idx]+=item.y;
    }
    for(int i=1;i<=all.size();i++) s[i]=s[i-1]+num[i];
    for(auto item:query)
    {
        int l=find(item.x),r=find(item.y);
        cout << s[r]-s[l-1] << endl;
    }
    return 0;
}
```

### 解题思路

>**离散化**

> 将输入的所有下标进行排序，进行离散化操作，并通过前缀和求出离散化后对应两个下标的数据总和；