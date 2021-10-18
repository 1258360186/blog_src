---
title: Windows 线程同步之 Mutex
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
categories:
  - C/C++ 多线程编程精髓
date: 2021-10-18 13:25:45
---
#### 音乐小港
{% meting "1458717405" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Windows 线程同步之 Mutex
`Mutex`（ 互斥量）采用的是英文 `Mutual Exclusive`（互相排斥之意）的缩写。见名知义，Windows 中的 `Mutex` 对象在同一个时刻最多只能属于一个线程，当然也可以不属于任何线程，获得 `Mutex` 对象的线程成为该 `Mutex` 的拥有者（`owner`）。我们可以在创建 `Mutex` 对象时设置 `Mutex` 是否属于创建它的线程，其他线程如果希望获得该 `Mutex`，则可以调用 `WaitForSingleObject` 进行申请。创建 `Mutex` 的 API 是 `CreateMutex`，其函数签名如下：
```
HANDLE CreateMutex(
      LPSECURITY_ATTRIBUTES lpMutexAttributes,
      BOOL                  bInitialOwner,
      LPCTSTR               lpName
);
```
参数和返回值说明：

- 参数 `lpMutexAttributes` 用法同 `CreateEvent`，上面已经介绍过了，一般设置为 `NULL`；
- 参数 `bInitialOwner`，设置调用 `CreateMutex` 的线程是否立即拥有该 `Mutex` 对象，`TRUE` 拥有，`FALSE` 不拥有，不拥有时，其他线程调用 `WaitForSingleObject` 可以获得该 `Mutex` 对象；
- 参数 `lpName`，`Mutex` 对象的名称，`Mutex` 对象和 `Event` 对象一样，也可以通过名称在多个线程之间共享，如果不需要名称则可以将该参数设置为 `NULL`，根据是否具有名称 `Mutex` 对象分为`具名 Mutex` 和`匿名 Mutex`；
- 返回值，如果函数调用成功返回 Mutex 的句柄，调用失败返回` NULL`。

当一个线程不再需要该 `Mutex`，可以使用 `ReleaseMutex` 函数释放 `Mutex`，让其他需要等待该 `Mutex` 的线程有机会获得该 `Mutex，ReleaseMutex` 的函数签名如下：
```
BOOL ReleaseMutex(HANDLE hMutex);
```
参数 `hMutex` 即需要释放所有权的 `Mutex` 对象句柄。

我们来看一段具体的实例代码：
```
#include <Windows.h>
#include <string>
#include <iostream>

HANDLE      g_hMutex = NULL;
int         g_iResource = 0;

DWORD __stdcall WorkerThreadProc(LPVOID lpThreadParameter)
{
    DWORD dwThreadID = GetCurrentThreadId();
    while (true)
    {
        if (WaitForSingleObject(g_hMutex, 1000) == WAIT_OBJECT_0)
        {
            g_iResource++;
            std::cout << "Thread: " << dwThreadID << " becomes mutex owner, ResourceNo: " << g_iResource  << std::endl;
            ReleaseMutex(g_hMutex);
        }
        Sleep(1000);
    }

    return 0;
}

int main()
{
    //创建一个匿名的 Mutex 对象，默认情况下主线程不拥有该 Mutex
    g_hMutex = CreateMutex(NULL, FALSE, NULL);

    HANDLE hWorkerThreads[5]; 
    for (int i = 0; i < 5; ++i)
    {
        hWorkerThreads[i] = CreateThread(NULL, 0, WorkerThreadProc, NULL, 0, NULL);
    }

    for (int i = 0; i < 5; ++i)
    {
        //等待工作线程退出
        WaitForSingleObject(hWorkerThreads[i], INFINITE);
        CloseHandle(hWorkerThreads[i]);
    }

    CloseHandle(g_hMutex);
    return 0;
}
```
在上述代码中，主线程创建一个 `Mutex`，并且设置不拥有它，然后五个工作线程去竞争获得这个 `Mutex` 的使用权，拿到这个 `Mutex` 之后就可以操作共享资源 `g_iResource` 了，程序的执行效果是五个工作线程随机获得该资源的使用权：

![multithreaded12](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded12.1k981ygzp0ps.jpg)

互斥体对象的排他性，有点类似于公共汽车上的座位，如果一个座位已经被别人占用，其他人需要等待，如果该座位没人坐，则其他人“先到先得”。当你不需要使用的时候，要把座位腾出来让其他有需要的人使用。假设某个线程在退出后，仍然没有释放其持有的 `Mutex` 对象，这时候使用 `WaitForSingleObject` 等待该 `Mutex` 对象的线程，也会立即返回，返回值是 `WAIT_ABANDONED`，表示该 `Mutex` 处于废弃状态（`abandoned`），处于废弃状态的 `Mutex` 不能再使用，其行为是未定义的。

通过上面的分析，我们知道 Windows `Mutex` 对象存在一个 `owner` 的概念，即哪个线程获取了该 `Mutex` 即成为该线程的 `owner`，事实上 `Mutex` 对象有一个内部属性记录了 `owner`，只不过微软没有公开而已。如果对 `Mutex` 内部实现原理感兴趣，可以阅读“开源版的 Windows” 的源码——`ReactOS`，其主页如下：https://reactos.org/。


---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)