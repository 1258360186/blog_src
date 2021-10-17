---
title: 区间合并
comments: true
mathjax: true
tags:
  - C++
  - 基础算法
categories:
  - 算法基础
date: 2021-10-13 17:45:44
---
#### 音乐小港
{% meting "28464997" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
##  区间合并

### 题目

给定 $n$ 个区间 $[l_i,r_i]$，要求合并所有有交集的区间。

注意如果在端点处相交，也算有交集。

输出合并完成后的区间个数。

例如：[1,3]和[2,6]可以合并为一个区间[1,6]。

### 输入格式

第一行包含整数n。

接下来n行，每行包含两个整数 l 和 r。

### 输出格式

共一行，包含一个整数，表示合并区间完成后的区间个数。

### 数据范围

$1≤n≤100000$,
−$10^9≤l_i≤r_i≤10^9$

### 输入样例

```
5
1 2
2 4
5 6
7 8
7 9
```

### 输出样例

```
3
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
int n;
vector<PII> f;

int merge(auto &f)
{
    vector<PII> g;
    int l=-2e9,r=-2e9;
    for(auto item:f)
    {
        if(r<item.x)
        {
            if(l!=-2e9) g.push_back({l,r});
            l=item.x,r=item.y;
        }
        else r=max(r,item.y);
    }
    g.push_back({l,r});
    return g.size();
}

int main()
{
    cin >> n;
    while(n--)
    {
        int l,r;
        cin >> l >> r;
        f.push_back({l,r});
    }
    sort(f.begin(),f.end());
    int res = merge(f);
    cout << res << endl;
    return 0;
}
```

### 解题思路

>**区间合并**

> 将所有区间从小到大排序；
>
> 若后一个区间的前项大于当前区间的后项则将当前区间保存；
>
> 若后一个区间的前项小于或等于则当前区间的后项为两者的较后边者；
>
> 最后一个区间无法存储，额外存储一下；