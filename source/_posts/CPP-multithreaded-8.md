---
title: Windows 线程同步之 Semaphore
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
categories:
  - C/C++ 多线程编程精髓
date: 2021-10-18 13:31:09
---
#### 音乐小港
{% meting "1423219826" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Windows 线程同步之 Semaphore
`Semaphore` 也是 Windows 多线程同步常用的对象之一，与上面介绍的 `Event`、`Mutex` 不同，信号量存在一个资源计数的概念，`Event` 对象虽然可以同时唤醒多个线程，但是它不能精确地控制同时唤醒指定数目的线程，而 `Semaphore` 可以。创建 `Semaphore` 对象的 API 函数签名如下：
```
HANDLE CreateSemaphore(
      LPSECURITY_ATTRIBUTES lpSemaphoreAttributes,
      LONG                  lInitialCount,
      LONG                  lMaximumCount,
      LPCTSTR               lpName
);
```
参数和返回值介绍：
- 参数 `lpSemaphoreAttributes` 指定了 `Semaphore` 对象的安全属性，一般设置为 `NULL` 使用默认安全属性；
- 参数 `lInitialCount` 指定初始可用资源数量，假设初始资源数量为 2，如果有 5 个线程正在调用 `WaitForSingleObject` 函数等待该信号量，则有 2 个线程会被唤醒，每调用一次 `WaitForSingleObject` 获得 `Semaphore` 对象，该对象的资源计数会减少一个。
- 参数 `lMaximumCount` 最大资源数量上限，如果使用 `ReleaseSemaphore` 不断增加资源计数，资源数量最大不能超过这个值，这个值必须设置大于 0。
- 参数 lpName 指定 `Semaphore` 对象的名称，`Semaphore` 对象也是可以通过名称跨进程共享的，如果不需要设置名称可以将该参数设置为 `NULL`，设置了名称的 `Semaphore` 对象被称为具名信号量，反之叫匿名信号量。
- 返回值：函数调用成功返回 `Semaphore` 对象的句柄，反之返回 `NULL`。
如果需要增加信号量的资源计数个数，可以使用 `ReleaseSemaphore` 函数，其签名如下：
```
BOOL ReleaseSemaphore(
      HANDLE hSemaphore,
      LONG   lReleaseCount,
      LPLONG lpPreviousCount
);
```
- 参数 `hSemaphore` 是需要操作的信号量句柄；
- 参数 `lReleaseCount`，需要增加的资源数量；
- 参数 `lpPreviousCount` 是一个 `long` 型（32 位系统上 4 个字节）的指针，函数执行成功后，返回是上一次资源的数量，如果用不到该参数，可以设置为 `NULL`。

信号量的使用方式类似，根据当前资源的数量按需分配资源消费者，资源消费者会让资源数量减少，如果资源数量减少为 0，消费者将全部处于挂起状态；当有新的资源来到时，消费者将继续被唤醒进行处理。

假设现在有一个即时通讯的程序，网络线程不断从网络上收到一条条聊天消息，其他 4 个消息处理线程需要对收到的聊天信息进行加工。由于我们需要根据当前消息的数量来唤醒其中 4 个工作线程中的一个或多个，这正是信号量使用的典型案例，代码如下：
```
#include <Windows.h>
#include <string>
#include <iostream>
#include <list>
#include <time.h>

HANDLE                  g_hMsgSemaphore = NULL;
std::list<std::string>  g_listChatMsg;
//保护 g_listChatMsg 的临界区对象
CRITICAL_SECTION        g_csMsg;

DWORD __stdcall NetThreadProc(LPVOID lpThreadParameter)
{
    int nMsgIndex = 0;
    while (true)
    {
        EnterCriticalSection(&g_csMsg);
        //随机产生1～4条消息
        int count = rand() % 4 + 1;
        for (int i = 0; i < count; ++i)
        {
            nMsgIndex++;
            SYSTEMTIME st;
            GetLocalTime(&st);
            char szChatMsg[64] = { 0 };
            sprintf_s(szChatMsg, 64, "[%04d-%02d-%02d %02d:%02d:%02d:%03d] A new msg, NO.%d.",
                st.wYear,
                st.wMonth,
                st.wDay,
                st.wHour,
                st.wMinute,
                st.wSecond,
                st.wMilliseconds,
                nMsgIndex);
            g_listChatMsg.emplace_back(szChatMsg);
        }   
        LeaveCriticalSection(&g_csMsg);

        //增加 count 个资源数量
        ReleaseSemaphore(g_hMsgSemaphore, count, NULL);
    }// end while-loop

    return 0;
}

DWORD __stdcall ParseThreadProc(LPVOID lpThreadParameter)
{
    DWORD dwThreadID = GetCurrentThreadId();
    std::string current;
    while (true)
    {
        if (WaitForSingleObject(g_hMsgSemaphore, INFINITE) == WAIT_OBJECT_0)
        {
            EnterCriticalSection(&g_csMsg);
            if (!g_listChatMsg.empty())
            {
                current = g_listChatMsg.front();
                g_listChatMsg.pop_front();
                std::cout << "Thread: " << dwThreadID << " parse msg: " << current << std::endl;
            }         
            LeaveCriticalSection(&g_csMsg);
        }
    }

    return 0;
}

int main()
{
    //初始化随机数种子
    srand(time(NULL));
    InitializeCriticalSection(&g_csMsg);

    //创建一个匿名的 Semaphore 对象，初始资源数量为 0
    g_hMsgSemaphore = CreateSemaphore(NULL, 0, INT_MAX, NULL);

    HANDLE hNetThread = CreateThread(NULL, 0, NetThreadProc, NULL, 0, NULL);

    HANDLE hWorkerThreads[4];
    for (int i = 0; i < 4; ++i)
    {
        hWorkerThreads[i] = CreateThread(NULL, 0, ParseThreadProc, NULL, 0, NULL);
    }

    for (int i = 0; i < 4; ++i)
    {
        //等待工作线程退出
        WaitForSingleObject(hWorkerThreads[i], INFINITE);
        CloseHandle(hWorkerThreads[i]);
    }

    WaitForSingleObject(hNetThread, INFINITE);
    CloseHandle(hNetThread);

    CloseHandle(g_hMsgSemaphore);

    DeleteCriticalSection(&g_csMsg);
    return 0;
}
```
在上述代码中，网络线程每次随机产生 1 ～ 4 个聊天消息放入消息容器 `g_listChatMsg` 中，然后根据当前新产生的消息数目调用 `ReleaseSemaphore` 增加相应的资源计数，这样就有相应的处理线程被唤醒，从容器 `g_listChatMsg` 中取出消息进行处理。

> 注意：由于会涉及到多个线程操作消息容器 `g_listChatMsg`，这里使用了一个临界区对象 `g_csMsg` 对其进行保护。

程序执行效果如下：
```
//这里截取输出中间部分...输出太多，部分结果省略
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.26.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.27.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.28.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.29.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.30.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.31.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.32.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.33.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.34.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.35.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.36.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.37.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.38.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.39.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.40.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.41.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.42.
Thread: 3704 parse msg: [2019-01-20 16:31:47:568] A new msg, NO.43.
Thread: 3704 parse msg: [2019-01-20 16:31:47:569] A new msg, NO.44.
Thread: 3704 parse msg: [2019-01-20 16:31:47:569] A new msg, NO.45.
Thread: 3704 parse msg: [2019-01-20 16:31:47:569] A new msg, NO.46.
Thread: 3704 parse msg: [2019-01-20 16:31:47:569] A new msg, NO.47.
Thread: 5512 parse msg: [2019-01-20 16:31:47:569] A new msg, NO.48.
Thread: 6676 parse msg: [2019-01-20 16:31:47:569] A new msg, NO.49.
Thread: 6676 parse msg: [2019-01-20 16:31:47:569] A new msg, NO.50.
```
总结起来，Semaphore 与上面介绍的 Event、Mutex 不一样，由于存在资源计数的概念，可以精准地控制同时唤醒几个等待的线程。

---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)