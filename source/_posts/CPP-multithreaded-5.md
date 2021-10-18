---
title: Windows 线程资源同步之临界区
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
  - GDB
categories:
  - C/C++ 多线程编程
date: 2021-10-18 12:59:12
---
#### 音乐小港
{% meting "552448704" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Windows 线程资源同步之临界区
前面章节介绍了多线程编程的一些基础内容，从本讲开始，我将系统地介绍一遍 Windows 和 Linux 操作系统下各种常用的多线程资源同步对象。

在开始介绍 Windows 多线程资源同步之前，我们来介绍两个重要的 Windows API 函数 `WaitForSingleObject` 和 `WaitForMultipleObjects`，Windows 上所有多线程同步对象基本上都是通过这两个函数完成的，前者只能一次操作一个资源同步对象，后者可以同时操作多个资源同步对象。

## WaitForSingleObject 与 WaitForMultipleObjects 函数
先来说 `WaitForSingleObject`，这个函数的签名是：
```
DWORD WaitForSingleObject(HANDLE hHandle, DWORD dwMilliseconds);
```
这个函数的作用是等待一个内核对象，在 Windows 系统上一个内核对象通常使用其句柄来操作，参数 `hHandle` 即需要等待的内核对象，参数 `dwMilliseconds` 是等待这个内核对象的最大时间，时间单位是毫秒，其类型是 `DWORD`，这是一个 `unsigned long` 类型。如果我们需要无限等待下去，可以将这个参数值设置为 `INFINITE` 宏。

在 Windows 上可以调用 `WaitForSingleObject` 等待的常见对象如下表所示：

|可以被等待的对象|	等待对象成功的含义|	对象类型|
|---|---|---|
|线程|	等待线程结束|	HANDLE|
|Process|	等待进程结束|	HANDLE|
|Event（事件）|	等待 Event 有信号|	HANDLE|
|Mutex (互斥体)|	等待持有 Mutex 的线程释放该 Mutex，等待成功，拥有该 Mutex|	HANDLE|
|Semaphore（信号量）|	等待该 Semaphore 对象有信号|	HANDLE|

上面介绍的等待线程对象上文中已经详细介绍过了，这里不再重复了，等待进程退出与等待线程退出类似，也不再赘述。下文中我们将详细介绍 `Event`、`Mutex`、`Semaphore` 这三种类型的资源同步对象，这里我们先接着介绍 `WaitForSingleObject` 函数的用法，该函数的返回值一般有以下类型：

- `WAIT_FAILED`，表示 `WaitForSingleObject` 函数调用失败了，可以通过 `GetLastError()` 函数得到具体的错误码；
- `WAIT_OBJECT_0`，表示 `WaitForSingleObject` 成功“等待”到设置的对象；
- `WAIT_TIMEOUT`，等待超时；
- `WAIT_ABANDONED`，当等待的对象是 `Mutex` 类型时，如果持有该 `Mutex` 对象的线程已经结束，但是没有在结束前释放该 `Mutex`，此时该 `Mutex` 已经处于废弃状态，其行为是未知的，不建议再使用。

`WaitForSingleObject` 如其名字一样，只能“等待”单个对象，如果需要同时等待多个对象可以使用 `WaitForMultipleObjects`，除了对象的数量变多了，其用法基本上和 `WaitForSingleObject` 一样。 `WaitForMultipleObjects` 函数签名如下：
```
DWORD WaitForMultipleObjects(
    DWORD        nCount,
    const HANDLE *lpHandles,
    BOOL         bWaitAll,
    DWORD        dwMilliseconds
);
```
参数 `lpHandles` 是需要等待的对象数组指针，参数 `nCount` 指定了该数组的长度，参数 `bWaitAll` 表示是否等待数组 `lpHandles` 所有对象有“信号”，取值为 `TRUE` 时，`WaitForMultipleObjects` 会等待所有对象有信号才会返回，取值为 `FALSE` 时，当其中有一个对象有信号时，立即返回，此时其返回值表示哪个对象有信号。

在参数 `bWaitAll` 设置为 `FALSE` 的情况下， 除了上面介绍的返回值是 `WAITFAILED` 和 `WAITTIMEOUT` 以外，返回值还有另外两种情形（分别对应 `WaitForSingleObject` 返回值是 `WAIT_OBJECT_0` 和 `WAIT_ABANDONED` 两种情形）：

- `WAIT_OBJECT_0` to (`WAIT_OBJECT_0 + nCount– 1`)，举个例子，假设现在等待三个对象 A1、A2、A3，它们在数组 `lpHandles` 中的下标依次是 0、1、2，某次 `WaitForMultipleObjects` 返回值是 `Wait_OBJECT_0 + 1`，则表示对象 A2 有信号，导致 `WaitForMultipleObjects` 调用成功返回。

伪码如下：
```
  HANDLE waitHandles[3];
  waitHandles[0] = hA1Handle;
  waitHandles[1] = hA2Handle;
  waitHandles[2] = hA3Handle;

  DWORD dwResult = WaitForMultipleObjects(3, waitHandles, FALSE, 3000);
  switch(dwResult)
  {
      case WAIT_OBJECT_0 + 0:
          //A1 有信号
          break;

      case WAIT_OBJECT_0 + 1:
          //A2 有信号
          break;

      case WAIT_OBJECT_0 + 2:
          //A3 有信号
          break;

      default:
          //出错或超时
          break;
  }
```
- `WAIT_ABANDONED_0` to (`WAIT_ABANDONED_0 + nCount– 1`)，这种情形与上面的使用方法相同，通过 `nCount - 1` 可以知道是等待对象数组中哪个对象始终没有被其他线程释放使用权。

> 这里说了这么多理论知识，读者将在下文介绍的 Windows 常用的资源同步对象章节中看到具体的示例代码。

## Windows 的临界区对象
在所有的 Windows 资源同步对象中，`CriticalSection` （临界区对象，有些书上翻译成“关键段”）是最简单易用的，从程序的术语来说，它防止多线程同时执行其保护的那段代码（`临界区代码`），即临界区代码某一时刻只允许一个线程去执行，示意图如下：

![multithreaded9](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded9.1nxd1pd0sxhc.jpg)

Windows 没有公开 `CriticalSection` 数据结构的定义，我们一般使用如下五个 API 函数操作临界区对象：
```
void InitializeCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
void DeleteCriticalSection(LPCRITICAL_SECTION lpCriticalSection);

BOOL TryEnterCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
void EnterCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
void LeaveCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
```
`InitializeCriticalSection` 和 `DeleteCriticalSection` 用于初始化和销毁一个 `CRITICAL_SECTION` 对象；位于`EnterCriticalSection` 和 `LeaveCriticalSection` 之间的代码即临界区代码；调用 `EnterCriticalSection` 的线程会尝试“进入“临界区，如果进入不了，则会阻塞调用线程，直到成功进入或者超时；`TryEnterCriticalSection` 会尝试进入临界区，如果可以进入，则函数返回 `TRUE` ，如果无法进入则立即返回不会阻塞调用线程，函数返回 `FALSE`。`LeaveCriticalSection` 函数让调用线程离开临界区，离开临界区以后，临界区的代码允许其他线程调用 `EnterCriticalSection` 进入。

> `EnterCriticalSection` 超时时间很长，可以在注册表 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager` 这个位置修改参数 `CriticalSectionTimeout` 的值调整，当然实际开发中我们从来不会修改这个值，如果你的代码等待时间较长最终超时，请检查你的逻辑设计是否合理。

我们来看一段实例代码：
```
#include <Windows.h>
#include <list>
#include <iostream>
#include <string>
 
CRITICAL_SECTION       g_cs;
int                    g_number = 0;

DWORD __stdcall WorkerThreadProc(LPVOID lpThreadParameter)
{
    DWORD dwThreadID = GetCurrentThreadId();
    
    while (true)
    {
        EnterCriticalSection(&g_cs);
          std::cout << "EnterCriticalSection, ThreadID: " << dwThreadID << std::endl;
        g_number++;
        SYSTEMTIME st;
        //获取当前系统时间
        GetLocalTime(&st);
        char szMsg[64] = { 0 };
        sprintf(szMsg, 
                "[%04d-%02d-%02d %02d:%02d:%02d:%03d]NO.%d, ThreadID: %d.", 
                st.wYear, st.wMonth, st.wDay, 
                  st.wHour, st.wMinute, st.wSecond, st.wMilliseconds, 
                g_number, dwThreadID);

        std::cout << szMsg << std::endl;
        std::cout << "LeaveCriticalSection, ThreadID: " << dwThreadID << std::endl;
        LeaveCriticalSection(&g_cs);

        //睡眠1秒
        Sleep(1000);
    }

    return 0;
}

int main()
{
    InitializeCriticalSection(&g_cs);

    HANDLE hWorkerThread1 = CreateThread(NULL, 0, WorkerThreadProc, NULL, 0, NULL);
    HANDLE hWorkerThread2 = CreateThread(NULL, 0, WorkerThreadProc, NULL, 0, NULL);

    WaitForSingleObject(hWorkerThread1, INFINITE);
    WaitForSingleObject(hWorkerThread2, INFINITE);

    //关闭线程句柄
    CloseHandle(hWorkerThread1);
    CloseHandle(hWorkerThread2);

    DeleteCriticalSection(&g_cs);

    return 0;
}
```
上述程序执行输出结果如下：
```
EnterCriticalSection, ThreadID: 1224
[2019-01-19 22:25:41:031]NO.1, ThreadID: 1224.
LeaveCriticalSection, ThreadID: 1224
EnterCriticalSection, ThreadID: 6588
[2019-01-19 22:25:41:031]NO.2, ThreadID: 6588.
LeaveCriticalSection, ThreadID: 6588
EnterCriticalSection, ThreadID: 6588
[2019-01-19 22:25:42:031]NO.3, ThreadID: 6588.
LeaveCriticalSection, ThreadID: 6588
EnterCriticalSection, ThreadID: 1224
[2019-01-19 22:25:42:031]NO.4, ThreadID: 1224.
LeaveCriticalSection, ThreadID: 1224
EnterCriticalSection, ThreadID: 1224
[2019-01-19 22:25:43:031]NO.5, ThreadID: 1224.
LeaveCriticalSection, ThreadID: 1224
EnterCriticalSection, ThreadID: 6588
[2019-01-19 22:25:43:031]NO.6, ThreadID: 6588.
LeaveCriticalSection, ThreadID: 6588
EnterCriticalSection, ThreadID: 1224
[2019-01-19 22:25:44:031]NO.7, ThreadID: 1224.
LeaveCriticalSection, ThreadID: 1224
EnterCriticalSection, ThreadID: 6588
[2019-01-19 22:25:44:031]NO.8, ThreadID: 6588.
LeaveCriticalSection, ThreadID: 6588
```
在上述代码中我们新建两个工作线程，线程函数都是 `WorkerThreadProc`。线程函数在 15 行调用 `EnterCriticalSection` 进入临界区，在 30 行调用 `LeaveCriticalSection` 离开临界区，16 ～ 29 行之间的代码即临界区的代码，这段代码由于受到临界区对象 `g_cs` 的保护，因为每次只允许一个工作线程执行这段代码。虽然临界区代码中有多个输出，但是这些输出一定都是连续的，不会出现交叉输出的结果。

细心的读者会发现上述输出中存在同一个的线程连续两次进入临界区，这是有可能的。也就是说，当其中一个线程离开临界区，即使此时有其他线程在这个临界区外面等待，由于线程调度的不确定性，此时正在等待的线程也不会有先进入临界区的优势，它和刚离开这个临界区的线程再次竞争进入临界区是机会均等的。我们来看一张图：

![multithreaded10](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded10.4u16k5jia900.jpg)

上图中我们将线程函数的执行流程绘制成一个流程图，两个线程竞争进入临界区可能存在如下情形，为了表述方便，将线程称为 A、B。

- 情形一：**线程 A** 被唤醒获得 CPU 时间片进入临界区，执行流程 ①，然后执行临界区代码输出 → **线程 B** 获得 CPU 时间片，执行流程 ②，然后失去 CPU 时间片进入休眠 → **线程 A** 执行完临界区代码离开临界区后执行流程 ⑤，然后失去 CPU 时间片进入休眠 → **线程 B** 被唤醒获得 CPU 时间片执行流程 ③、①，然后执行临界区代码输出。

  这种情形下，线程 A 和线程 B 会轮流进入临界区执行代码。
- 情形二：**线程 A** 被唤醒获得 CPU 时间片进入临界区，执行流程 ①，然后执行临界区代码输出 → **线程 B** 获得 CPU 时间片，执行流程 ③，然后执行流程 ② 在临界区外面失去 CPU 时间片进入休眠 → **线程 A** 执行完临界区代码离开临界区后执行流程 ④、① 。

  这种情形下，会出现某个线程连续两次甚至更多次的进入临界区执行代码。

如果某个线程在尝试进入临界区时因无法阻塞而进入睡眠状态，当其他线程离开这个临界区后，之前因为这个临界区而阻塞的线程可能会被唤醒进行再次竞争，也可能不被唤醒。但是存在这样一种特例，假设现在存在两个线程 A 和 B，**线程 A** 离开临界区的线程再也不需要再次进入临界区，那么**线程 B** 在被唤醒时一定可以进入临界区。**线程 B** 从睡眠状态被唤醒，这涉及到一次线程的切换，有时候这种开销是不必要的，我们可以让 B 简单地执行一个循环等待一段时间后去进去临界区，而不是先睡眠再唤醒，与后者相比，执行这个循环的消耗更小。这就是所谓的“自旋”，在这种情形下，Windows 提供了另外一个初始化临界区的函数 `InitializeCriticalSectionAndSpinCount`，这个函数比 `InitializeCriticalSection` 多一个自旋的次数：
```
BOOL InitializeCriticalSectionAndSpinCount(
      LPCRITICAL_SECTION lpCriticalSection,
      DWORD              dwSpinCount
);
```
参数 `dwSpinCount` 是自旋的次数，利用自旋来代替让 CPU 进入睡眠和再次被唤醒，消除线程上下文切换带来的消耗，提高效率。当然，在实际开发中这种方式是靠不住的，线程调度是操作系统内核的策略，应用层上的应用不应该假设线程的调度策略是按预想的来执行。但是理解线程与临界区之间的原理有利于编写出更高效的应用来。

需要说明的是，临界区对象通过保护一段代码不被多个线程同时执行，进而来保证多个线程之间读写一个对象是安全的。由于同一时刻只有一个线程可以进入临界区，因此这种对资源的操作是排他的，即对于同一个临界区对象，不会出现多个线程同时操作该资源，哪怕是资源本身可以在同一时刻被多个线程进行操作，如多个线程对资源进行读操作，这就带来了效率问题。

我们一般将进入临界区的线程称为该临界区的拥有者（`owner`）——临界区持有者。

最后，为了避免死锁，`EnterCriticalSection` 和 `LeaveCriticalSection` 需要成对使用，尤其是在具有多个出口的函数中，记得在每个分支处加上 `LeaveCriticalSection`。伪码如下：
```
void someFunction()
{
    EnterCriticalSection(&someCriticalSection);
    if (条件A)
    {
        if (条件B)
        {
            LeaveCriticalSection(&someCriticalSection);
            //出口1
            return;
        }

        LeaveCriticalSection(&someCriticalSection);
        //出口2
        return;
    }

    if (条件C)
    {
        LeaveCriticalSection(&someCriticalSection);
        // 出口3
        return;
    }

    if (条件C)
    {
        LeaveCriticalSection(&someCriticalSection);
        // 出口4
        return;
    }
}
```
上述代码中，为了能让临界区对象被正常的释放，在函数的每个出口都加上了 `LeaveCriticalSection` 调用，如果函数的出口非常多，这样的代码太难维护了。因此一般建议使用 `RAII` 技术将临界区 API 封装成对象，该对象在函其作用域内进入临界区，在出了其作用域后自动离开临界区，示例代码如下：
```
class CCriticalSection
{
public:
    CCriticalSection(CRITICAL_SECTION& cs) : mCS(cs)
    {
        EnterCriticalSection(&mCS);
    }

    ~CCriticalSection()
    {
        LeaveCriticalSection(&mCS);
    }

private:
    CRITICAL_SECTION& mCS;
};
```
利用 `CCriticalSection` 类，我们可以对上述伪码进行优化：
```
void someFunction()
{
    CCriticalSection autoCS(someCriticalSection);
    if (条件A)
    {
        if (条件B)
        { 
            //出口1
            return;
        }

        //出口2
        return;
    }

    if (条件C)
    {      
        // 出口3
        return;
    }

    if (条件C)
    {        
        // 出口4
        return;
    }
}
```
上述代码中由于变量 `autoCS` 会在出了函数作用域后调用其析构函数，在析构函数中调用 `LeaveCriticalSection` 自动离开临界区。

## 总结
本讲介绍了 `WaitForSingleObject` 和 `WaitForMultipleObjects` 这两个重要的 Windows API 函数，同时介绍了 Windows 上第一个线程同步对象——临界区，为了避免因函数有多个出口造成的编码疏漏，我们介绍了使用 `RAII` 封装临界区对象的方法。临界区对象是 Windows 系统多线程资源同步最常用的对象之一。



---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)