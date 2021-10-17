---
title: Linux C++ 开发 第二讲
comments: true
mathjax: true
tags:
  - Linux
  - C++ 开发
categories:
  - 基于VSCode和CMake实现C/C++开发 | Linux篇
date: 2021-10-10 17:24:59
---
#### 音乐小港
{% meting "1371714110" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# **Linux C++ 开发**

## **第二讲: 开发环境搭建**

### **2.1 编译器，调试器安装**
- 安装GCC，GDB
  ```
  sudo apt update
  # 通过以下命令安装编译器和调试器
  sudo apt install build-essential gdb
  ```
- 安装成功确认
  ```
  # 以下命令确认每个软件是否安装成功
  # 如果成功，则显示版本号
  gcc --version
  g++ --version
  gdb --version
  ```

### **2.2 CMake安装**
- 安装cmake
  ```
  # 通过以下命令安装编译器和调试器
  sudo apt install cmake
  ```
- 安装成功确认
  ```
  # 确认是否安装成功
  # 如果成功，则显示版本号
  cmake --version
  ```

---
课程来源:[基于VSCode和CMake实现C/C++开发 | Linux篇](https://www.bilibili.com/video/BV1fy4y1b7TC)