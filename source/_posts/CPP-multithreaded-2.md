---
title: 如何等待线程结束
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
categories:
  - C/C++ 多线程编程精髓
date: 2021-10-18 12:31:43
---
#### 音乐小港
{% meting "1826142553" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# 如何等待线程结束
前面课程介绍了创建线程，既然线程可以创建，线程也应该可以结束。那如何等待一个线程结束呢？

实际项目开发中，我们常常会有这样一种需求，即一个线程需要等待另外一个线程执行完任务退出后再继续执行。这在 Linux 和 Windows 操作系统中都提供了相应的操作系统 API，我们来分别介绍一下。

## Linux 下等待线程结束
Linux 线程库提供了 `pthread_join` 函数，用来等待某线程的退出并接收它的返回值。这种操作被称为连接（`joining`），`pthread_join` 函数签名如下：
```
int pthread_join(pthread_t thread, void** retval);
```
- 参数 `thread`，需要等待的线程 id。
- 参数 `retval`，输出参数，用于接收等待退出的线程的退出码（`Exit Code`），线程退出码可以通过调用 `pthread_exit` 退出线程时指定，也可以在线程函数中通过 `return` 语句返回。
  ```
  #include <pthread.h>

  void pthread_exit(void* value_ptr);&nbsp;
  ```
参数 `value_ptr` 的值可以在 `pthread_join` 中拿到，没有可以设置为 `NULL`。

**`pthread_join` 函数等待其他线程退出期间会挂起等待的线程**，被挂起的线程不会消耗宝贵任何 CPU 时间片。直到目标线程退出后，等待的线程会被唤醒。

我们通过一个实例来演示一下这个函数的使用方法，实例功能如下：

程序启动时，开启一个工作线程，工作线程将当前系统时间写入文件中后退出，主线程等待工作线程退出后，从文件中读取出时间并显示在屏幕上。代码如下：
```
#include <stdio.h>
#include <string.h>
#include <pthread.h>

#define TIME_FILENAME "time.txt"

void* fileThreadFunc(void* arg)
{
    time_t now = time(NULL);
    struct tm* t = localtime(&now);
    char timeStr[32] = {0};
    snprintf(timeStr, 32, "%04d/%02d/%02d %02d:%02d:%02d", 
             t->tm_year+1900,
             t->tm_mon+1,
             t->tm_mday,
             t->tm_hour,
             t->tm_min,
             t->tm_sec);
    //文件不存在，则创建；存在，则覆盖
    FILE* fp = fopen(TIME_FILENAME, "w");
    if (fp == NULL)
    {
      printf("Failed to create time.txt.\n");
        return;
    }

    size_t sizeToWrite = strlen(timeStr) + 1;
    size_t ret = fwrite(timeStr, 1, sizeToWrite, fp);
    if (ret != sizeToWrite)
    {
        printf("Write file error.\n");
    }

    fclose(fp);
}

int main()
{
    pthread_t fileThreadID;
    int ret = pthread_create(&fileThreadID, NULL, fileThreadFunc, NULL);
    if (ret != 0)
    {
        printf("Failed to create fileThread.\n");
        return -1;
    }

    int* retval;
    pthread_join(fileThreadID, (void**)&retval);

    //使用r选项，要求文件必须存在
    FILE* fp = fopen(TIME_FILENAME, "r");
    if (fp == NULL)
    {
        printf("open file error.\n");
        return -2;
    }

    char buf[32] = {0};
    int sizeRead = fread(buf, 1, 32, fp);
    if (sizeRead == 0)
    {
      printf("read file error.\n");
      return -3;
    }

    printf("Current Time is: %s.\n", buf);

    return 0;
}
```
程序执行结果如下：
```
[root@localhost threadtest]# ./test
Current Time is: 2018/09/24 21:06:01.
```
## Windows 下等待线程结束
Windows 下使用 API `WaitForSingleObject` 或 `WaitForMultipleObjects` 函数，前者用于等待一个线程结束，后者可以同时等待多个线程结束。这是两个非常重要的函数，它们的作用不仅可以用于等待线程退出，还可以用于等待其他线程同步对象，本文后面的将详细介绍这两个函数。与 Linux 的 ``pthread_join`` 函数不同，Windows 的`WaitForSingleObject` 函数提供了可选择等待时间的精细控制。

这里我们仅演示等待线程退出。

`WaitForSingleObject` 函数签名如下：
```
DWORD WaitForSingleObject(HANDLE hHandle, DWORD dwMilliseconds);
```
- 参数 `hHandle` 是需要等待的对象的句柄，等待线程退出，传入线程句柄。
- 参数 `dwMilliseconds` 是需要等待的毫秒数，如果使用 `INFINITE` 宏，则表示无限等待下去。
- 返回值：该函数的返回值有点复杂，我们后面文章具体介绍。当 `dwMilliseconds` 参数使用 `INFINITE` 值，该函数会挂起当前等待线程，直到等待的线程退出后，等待的线程才会被唤醒，`WaitForSingleObject` 后的程序执行流继续执行。
我们将上面的 Linux 示例代码改写成 Windows 版本的：
```
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <Windows.h>

#define TIME_FILENAME "time.txt"

DWORD WINAPI FileThreadFunc(LPVOID lpParameters)
{
    time_t now = time(NULL);
    struct tm* t = localtime(&now);
    char timeStr[32] = { 0 };
    sprintf_s(timeStr, 32, "%04d/%02d/%02d %02d:%02d:%02d",
              t->tm_year + 1900,
              t->tm_mon + 1,
              t->tm_mday,
              t->tm_hour,
              t->tm_min,
              t->tm_sec);
    //文件不存在，则创建；存在，则覆盖
    FILE* fp = fopen(TIME_FILENAME, "w");
    if (fp == NULL)
    {
        printf("Failed to create time.txt.\n");
        return 1;
    }

    size_t sizeToWrite = strlen(timeStr) + 1;
    size_t ret = fwrite(timeStr, 1, sizeToWrite, fp);
    if (ret != sizeToWrite)
    {
        printf("Write file error.\n");
    }

    fclose(fp);

    return 2;
}


int main()
{
    DWORD dwFileThreadID;
    HANDLE hFileThread = CreateThread(NULL, 0, FileThreadFunc, NULL, 0, 
                                      &dwFileThreadID);
    if (hFileThread == NULL)
    {
        printf("Failed to create fileThread.\n");
        return -1;
    }

    //无限等待，直到文件线程退出，否则程序将一直挂起
    WaitForSingleObject(hFileThread, INFINITE);

    //使用r选项，要求文件必须存在
    FILE* fp = fopen(TIME_FILENAME, "r");
    if (fp == NULL)
    {
        printf("open file error.\n");
        return -2;
    }

    char buf[32] = { 0 };
    int sizeRead = fread(buf, 1, 32, fp);
    if (sizeRead == 0)
    {
        printf("read file error.\n");
        return -3;
    }

    printf("Current Time is: %s.\n", buf);

    return 0;
}
```
与 Linux 版本一样，我们得到类似的程序执行结果：

![multithreaded4](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded4.700vsr4a9js0.jpg)

## C++ 11 提供的等待线程结果函数
可以想到，`C++ 11` 的 `std::thread` 既然统一了 Linux 和 Windows 的线程创建函数，那么它应该也提供等待线程退出的接口，确实如此，`std::thread` 的 `join` 方法就是用来等待线程退出的函数。当然使用这个函数时，必须保证该线程还处于运行中状态，也就是说等待的线程必须是可以 “`join`”的，如果需要等待的线程已经退出，此时调用`join` 方法，程序会产生崩溃。因此，`C++ 11` 的线程库同时提供了一个 `joinable` 方法来判断某个线程是否可以 `join`，如果不确定线程是否可以“`join`”，可以先调用 `joinable` 函数判断一下是否需要等待。

还是以上面的例子为例，改写成 `C++11` 的代码：
```
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <thread>

#define TIME_FILENAME "time.txt"

void FileThreadFunc()
{
    time_t now = time(NULL);
    struct tm* t = localtime(&now);
    char timeStr[32] = { 0 };
    sprintf_s(timeStr, 32, "%04d/%02d/%02d %02d:%02d:%02d",
              t->tm_year + 1900,
              t->tm_mon + 1,
              t->tm_mday,
              t->tm_hour,
              t->tm_min,
              t->tm_sec);
    //文件不存在，则创建；存在，则覆盖
    FILE* fp = fopen(TIME_FILENAME, "w");
    if (fp == NULL)
    {
        printf("Failed to create time.txt.\n");
        return;
    }

    size_t sizeToWrite = strlen(timeStr) + 1;
    size_t ret = fwrite(timeStr, 1, sizeToWrite, fp);
    if (ret != sizeToWrite)
    {
        printf("Write file error.\n");
    }

    fclose(fp);
}

int main()
{
    std::thread t(FileThreadFunc);
    if (t.joinable())
        t.join();

    //使用 r 选项，要求文件必须存在
    FILE* fp = fopen(TIME_FILENAME, "r");
    if (fp == NULL)
    {
        printf("open file error.\n");
        return -2;
    }

    char buf[32] = { 0 };
    int sizeRead = fread(buf, 1, 32, fp);
    if (sizeRead == 0)
    {
        printf("read file error.\n");
        return -3;
    }

    printf("Current Time is: %s.\n", buf);

    return 0;
}
```
## 总结
在 A 线程等待 B 线程结束，相当于 A 线程和 B 线程在该点汇集（或连接），这就是 `join` 函数的语义来源，因此很多其他编程语言也使用 `join` 一词表示等待线程结束。

---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)