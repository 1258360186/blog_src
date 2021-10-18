---
title: Windows 线程资源同步之 Event
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
  - GDB
categories:
  - C/C++ 多线程编程
date: 2021-10-18 13:15:07
---
#### 音乐小港
{% meting "440403855" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Windows 线程资源同步之 Event
前面一讲我们介绍了 `WaitForSingleObject` 和 `WaitForMultipleObjects` 函数，它们是 Windows 多线程编程基础 API，所有资源同步对象都得和它们打交道；然后介绍了可以临界区对象，并且介绍了一个把临界区当自旋锁的扩展用法。本讲我们来介绍 Windows 最常用的多线程同步对象 —— `Event`。

## Event 使用方法
本节讨论的 `Event` 对象不是 Windows UI 事件驱动机制中的事件，而是多线程资源同步中的 `Event` 对象，它也是 Windows 内核对象之一。在 Windows 多线程程序设计中，使用频率较高，我们先来学习一下如何创建 `Event` 对象，然后逐步展开。创建 `Event` 的 Windows API 函数签名是：
```
HANDLE CreateEvent(
  LPSECURITY_ATTRIBUTES lpEventAttributes,
  BOOL                  bManualReset,
  BOOL                  bInitialState,
  LPCTSTR               lpName
);
```
参数和返回值的说明如下。

- 参数 `lpEventAttributes`，这个参数设置了 `Event` 对象的安全属性，Windows 中所有的内核对象都可以设置这个属性，我们一般设置为 `NULL`，即使用默认安全属性。
- 参数 `bManualReset`，这个参数设置 `Event` 对象受信（变成有信号状态）时的行为，当设置为 `TRUE` 时，表示需要手动调用 `ResetEvent` 函数去将 `Event` 重置成无信号状态；当设置为 `FALSE`，`Event` 事件对象受信后会自动重置为无信号状态。
- 参数 `bInitialState` 设置 `Event` 事件对象初始状态是否是受信的，`TRUE` 表示有信号，`FALSE` 表示无信号。
- 参数 `lpName` 可以设置 `Event` 对象的名称，如果不需要设置名称，可以将该参数设置为 `NULL`。一个 `Event` 对象根据是否设置了名称分为具名对象（具有名称的对象）和匿名对象。`Event` 对象是可以通过名称在不同进程之间共享的，通过这种方式共享很有用，后面我们会详细介绍。
- 返回值，如果成功创建 `Event` 对象返回对象的句柄，如果创建失败返回 `NULL`。

一个无信号的 `Event` 对象，我们可以通过 `SetEvent` 将其变成受信状态，`SetEvent` 的函数签名如下：
```
BOOL SetEvent(HANDLE hEvent);
```
我们将参数 `hEvent` 设置为需要设置信号的 `Event` 句柄即可。

同理，一个已经受信的 `Event` 对象，可以使用 `ResetEvent` 对象将其变成无信号状态，`ResetEvent` 的函数签名如下：
```
BOOL ResetEvent(HANDLE hEvent);
```
参数 `hEvent` 即我们需要重置的 Event 对象句柄。

说了这么多，来看一个具体的例子。假设现在有两个线程，其中一个是主线程，主线程等待工作线程执行某一项耗时的任务完成后，将任务结果显示出来。代码如下：
```
#include <Windows.h>
#include <string>
#include <iostream>

bool        g_bTaskCompleted = false;
std::string g_TaskResult;

DWORD __stdcall WorkerThreadProc(LPVOID lpThreadParameter)
{
    //使用 Sleep 函数模拟一个很耗时的操作
    //睡眠3秒
    Sleep(3000);
    g_TaskResult = "task completed";
    g_bTaskCompleted = true;

    return 0;
}

int main()
{
    HANDLE hWorkerThread = CreateThread(NULL, 0, WorkerThreadProc, NULL, 0, NULL); 
    while (true)
    {
        if (g_bTaskCompleted)
        {
            std::cout << g_TaskResult << std::endl;
            break;
        }        
        else
            std::cout << "Task is in progress..." << std::endl;
    }

    CloseHandle(hWorkerThread);
    return 0;
}
```
上述代码在主线程（`main` 函数）创建一个工作线程，然后主线程进入一个无限循环等待布尔变量 `gbTaskCompleted` 变成真值；工作线程（`WorkerThreadProc` 为线程函数）休眠 3 秒后将变量 `gbTaskCompleted` 设置为真后主线程得以退出循环，并打印代表条件满足后的结果。

程序执行结果如下图所示：

![multithreaded11](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded11.2ls5e8ckeoe0.jpg)

在上述代码中，主线程为了等待工作线程完成任务后获取结果，使用了一个循环去不断查询任务完成标识，这是很低效的一种做法，等待的线程（主线程）做了很多无用功，对 CPU 时间片也是一种浪费。我们使用 `Event` 对象来改写一下上述代码：
```
#include <Windows.h>
#include <string>
#include <iostream>

bool        g_bTaskCompleted = false;
std::string g_TaskResult;
HANDLE      g_hTaskEvent = NULL;

DWORD __stdcall WorkerThreadProc(LPVOID lpThreadParameter)
{
    //使用 Sleep 函数模拟一个很耗时的操作
    //睡眠3秒
    Sleep(3000);
    g_TaskResult = "task completed";
    g_bTaskCompleted = true;

    //设置事件信号
    SetEvent(g_hTaskEvent);

    return 0;
}

int main()
{
    //创建一个匿名的手动重置初始无信号的事件对象
    g_hTaskEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
    HANDLE hWorkerThread = CreateThread(NULL, 0, WorkerThreadProc, NULL, 0, NULL); 
    
    DWORD dwResult = WaitForSingleObject(g_hTaskEvent, INFINITE);
    if (dwResult == WAIT_OBJECT_0)
    {
        std::cout << g_TaskResult << std::endl;
    }
    
    CloseHandle(hWorkerThread);
    CloseHandle(g_hTaskEvent);
    return 0;
}
```
在上述代码中，主线程在工作线程完成任务之前会一直阻塞代码 29 行，没有任何消耗，当工作线程完成任务后调用 `SetEvent` 让事件对象受信，这样主线程会立即得到通知，从 `WaitForSingleObject` 返回，此时任务已经完成，就可以得到任务结果了。

在实际的开发中，可以利等待的时间去做一点其他的事情，在我们需要的时候去检测一下事件对象是否有信号即可。另外，`Event` 对象有两个显著的特点：

- 与临界区对象（以及接下来要介绍的 `Mutex` 对象）相比，`Event` 对象没有被谁持让持有者线程变成其 `owner` 这一说法，因此 `Event` 对象可以同时唤醒多个等待的工作线程。
- 手动重置的 `Event` 对象一旦变成受信状态，其信号不会丢失，也就是说当 `Event` 从无信号变成有信号时，即使某个线程当时没有调用 `WaitForSingleObject` 等待该 `Event` 对象受信，而是在这之后才调用 `WaitForSingleObject` ，仍然能检测到事件的受信状态，即不会丢失信号，而后面要介绍的条件变量就可能会丢失信号。
## Event 使用示例
蘑菇街开源的即时通讯 Teamtalk pc 版（[代码下载地址请点击这里](https://github.com/baloonwj/TeamTalk)）在使用 `socket` 连接服务器时，使用 `Event` 对象设计了一个超时做法。传统的做法是将 `socket` 设置为非阻塞的，调用完 `connect` 函数之后，调用 `select` 函数检测 `socket` 是否可写，在 `select` 函数里面设置超时时间。Teamtalk 的做法如下：
```
 //TcpClientModule_Impl.cpp 145行
 IM::Login::IMLoginRes* TcpClientModule_Impl::doLogin(CString &linkaddr, UInt16 port
    ,CString& uName,std::string& pass)
{
    //imcore::IMLibCoreConnect 中通过 connect 连接服务器
    m_socketHandle = imcore::IMLibCoreConnect(util::cStringToString(linkaddr), port);
    imcore::IMLibCoreRegisterCallback(m_socketHandle, this);
    if(util::waitSingleObject(m_eventConnected, 5000))
    {
        IM::Login::IMLoginReq imLoginReq;
        string& name = util::cStringToString(uName);
        imLoginReq.set_user_name(name);
        imLoginReq.set_password(pass);
        imLoginReq.set_online_status(IM::BaseDefine::USER_STATUS_ONLINE);
        imLoginReq.set_client_type(IM::BaseDefine::CLIENT_TYPE_WINDOWS);
        imLoginReq.set_client_version("win_10086");

        if (TCPCLIENT_STATE_OK != m_tcpClientState)
            return 0;

        sendPacket(IM::BaseDefine::SID_LOGIN, IM::BaseDefine::CID_LOGIN_REQ_USERLOGIN, ++g_seqNum
            , &imLoginReq);
        m_pImLoginResp->Clear();
        util::waitSingleObject(m_eventReceived, 10000);
    }

    return m_pImLoginResp;
}
```
`util::waitSingleObject` 即封装 API `WaitForSingleObject` 函数：
```
//utilCommonAPI.cpp 197行
BOOL waitSingleObject(HANDLE handle, Int32 timeout)
{
    int t = 0;
    DWORD waitResult = WAIT_FAILED;
    do
    {
        int timeWaiter = 500;
        t += timeWaiter;
        waitResult = WaitForSingleObject(handle, timeWaiter);
    } while ((WAIT_TIMEOUT == waitResult) && (t < timeout));

    return (WAIT_OBJECT_0 == waitResult);
}
```
等待的 `m_eventConnected` 对象即是一个 `Event` 类型：
```
//定义
HANDLE                            m_eventConnected;
//在 TcpClientModule_Impl 构造函数中初始化
//m_eventConnected = CreateEvent(NULL, FALSE, FALSE, NULL);
```
这个 `WaitForSingleObejct` 何时会返回呢？如果网络线程中 `connect` 函数可以正常连接服务器，会让 `m_eventConnected` 受信，这样 `WaitForSingleObejct` 函数就会返回了，接着组装登录数据包接着发数据。
```
void TcpClientModule_Impl::onConnectDone()
{
    m_tcpClientState = TCPCLIENT_STATE_OK;
    ::SetEvent(m_eventConnected);

    m_bDoReloginServerNow = FALSE;
    if (!m_pServerPingTimer)
    {
        _startServerPingTimer();
    }
}
```
归纳起来，这里利用了一个 `Event` 对象实现了一个同步登录的过程，网络连接最大超时事件设置成了 5000 毫秒（5 秒）：
```
util::waitSingleObject(m_eventConnected, 5000)
```
## 总结
`Event` 对象是 Windows 多线程最常用的同步对象之一，其特点是简单易用，如果多个线程都是等待一个 `Event` 对象受信，无法精确控制唤醒指定数量的线程，后面我们用信号量来解决该问题。


---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)