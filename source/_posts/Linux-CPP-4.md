---
title: Linux C++ 开发 第四讲
comments: true
mathjax: true
tags:
  - Linux
  - C++ 开发
categories:
  - 基于VSCode和CMake实现C/C++开发 | Linux篇
date: 2021-10-10 17:54:55
---
#### 音乐小港
{% meting "1344438011" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# **Linux C++ 开发**

## **第四讲: GDB调试器**

#### **前言**：
- **GDB(GNU Debugger)**是一个用来**调试C/C++程序**的功能强大的**调试器**，是Linux系统开发C/C++最常用的调试器
- 程序员可以**使用GDB来跟踪程序中的错误**，从而减少程序员的工作量。
- Linux 开发C/C++ 一定要熟悉 GDB
- **VSCode是通过调用GDB调试器来实现C/C++的调试工作的；**
> Windows 系统中，常见的集成开发环境（IDE），如 VS、VC等，它们内部已经嵌套了相应的调试器

#### **GDB主要功能**：
- 设置**断点**(断点可以是条件表达式)
- 使程序在指定的代码行上暂停执行，便于观察
- **单步**执行程序，便于调试
- 查看程序中变量值的变化
- 动态改变程序的执行环境
- 分析崩溃程序产生的core文件

### **4.1 常用调试命令参数**
调试开始：执行**gdb [exefilename]** ，进入gdb调试程序，其中exefilename为要调试的可执行文件名
```
## 以下命令后括号内为命令的简化使用，比如xiaobing run（r），直接输入命令 r 就代表命令run

$(gdb)help(h) # 查看命令帮助，具体命令查询在gdb中输入help + 命令

$(gdb)run(r) # 重新开始运行文件（run-text：加载文本文件，run-bin：加载件）

$(gdb)start # 单步执行，运行程序，停在第一行执行语句

$(gdb)list(l) # 查看原代码（list-n,从第n行开始查看代码。list+ 函数名：查数）

$(gdb)set # 设置变量的值

$(gdb)next(n) # 单步调试（逐过程，函数直接执行）

$(gdb)step(s) # 单步调试（逐语句：跳入自定义函数内部执行）

$(gdb)backtrace(bt) # 查看函数的调用的栈帧和层级关系

$(gdb)frame(f) # 切换函数的栈帧

$(gdb)info(i) # 查看函数内部局部变量的数值

$(gdb)finish # 结束当前函数，返回到函数调用点

$(gdb)continue(c) # 继续运行

$(gdb)print(p) # 打印值及地址

$(gdb)quit(q) # 退出gdb

$(gdb)break+num(b) # 在第num行设置断点

$(gdb)info breakpoints # 查看当前设置的所有断点

$(gdb)delete breakpoints num(d) # 删除第num个断点

$(gdb)display # 追踪查看具体变量值

$(gdb)undisplay # 取消追踪观察变量

$(gdb)watch # 被设置观察点的变量发生修改时，打印显示

$(gdb)i watch # 显示观察点

$(gdb)enable breakpoints # 启用断点

$(gdb)disable breakpoints # 禁用断点

$(gdb)x # 查看内存x/20xw 显示20个单元，16进制，4字节每单元

$(gdb)run argv[1] argv[2] # 调试时命令行传参

$(gdb)set follow-fork-mode child#Makefile项目管理：选择跟踪父子进程（fork()）
```
> **Tips**:
> 1. 编译程序时需要加上-g，之后才能用gdb进行调试：gcc -g main.c -o main
> 2. 回车键：重复上一命令

### **4.2 【实战】命令行调试**
给出一段简单代码，准备调试。
```
#include <iostream>
using namespace std;
int main(int argc,char **argv)
{
int N = 100;
int sum = 0;
int i = 1;
// calculate sum from 1 to 100
while (i <= N)
{
sum = sum + i;
i = i + 1;
}
cout << "sum = " << sum << endl;
cout << "The program is over." << endl;
return 0;
}
```

---
课程来源:[基于VSCode和CMake实现C/C++开发 | Linux篇](https://www.bilibili.com/video/BV1fy4y1b7TC)