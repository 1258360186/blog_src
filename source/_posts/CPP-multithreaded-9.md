---
title: 如何让程序只能启动一个实例
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
categories:
  - C/C++ 多线程编程精髓
date: 2021-10-18 13:35:45
---
#### 音乐小港
{% meting "1342827169" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# 如何让程序只能启动一个实例
前面介绍的 Windows `Event`、`Mutex`、`Semaphore` 对象其创建函数 `CreateX` 都可以给这些对象指定一个名字，有了名字之后这些线程资源同步对象就可以通过这个名字在不同进程之间共享。

在 Windows 系统上读者应该有这样的体验：有些程序无论双击其启动图标都只会启动一个，我们把这类程序叫做单实例程序（`Single Instance`）。可以利用命名的线程资源同步对象来实现这个效果，这里以互斥体为例。

示例代码如下：
```
int APIENTRY _tWinMain(_In_ HINSTANCE hInstance,
                     _In_opt_ HINSTANCE hPrevInstance,
                     _In_ LPTSTR    lpCmdLine,
                     _In_ int       nCmdShow)
{
    //...省略无关代码...

    if (CheckInstance())
    {
        HWND hwndPre = FindWindow(szWindowClass, NULL);
        if (IsWindow(hwndPre))
        {
            if (::IsIconic(hwndPre))
                ::SendMessage(hwndPre, WM_SYSCOMMAND, SC_RESTORE | HTCAPTION, 0);

            ::SetWindowPos(hwndPre, HWND_TOP, 0, 0, 0, 0, SWP_NOMOVE | SWP_NOSIZE | SWP_SHOWWINDOW | SWP_NOACTIVATE);
            ::SetForegroundWindow(hwndPre);
            ::SetFocus(hwndPre);
            return 0;
        }
    }

    //...省略无关代码
}
```
上述代码在 `WinMain` 函数开始处先检查是否已经运行起来的程序实例，如果存在，则找到运行中的实例程序主窗口并激活之，这就是读者看到最小化很多单例程序后双击该程序图标会重新激活最小化的程序的效果实现原理。

现在重点是 `CheckInstance()` 函数的实现：
```
bool CheckInstance()
{
    HANDLE hSingleInstanceMutex = CreateMutex(NULL, FALSE, _T("MySingleInstanceApp"));
    if (hSingleInstanceMutex != NULL)
    {
        if (GetLastError() == ERROR_ALREADY_EXISTS)
        {
            return true;
        }
    }

    return false;
}
```
我们来分析一下上述 `CheckInstance` 函数：

假设首次启动这个进程，这个进程会调用 `CreateMutex` 函数创建一个名称为“`MySingleInstanceApp`”的互斥体对象。当再次准备启动一份这个进程时，再次调用 `CreateMutex` 函数，由于该名称的互斥体对象已经存在，将会返回已经存在的互斥体对象地址，此时通过 `GetLastError()` 函数得到的错误码是 `ERROR_ALREADY_EXISTS` 表示该名称的互斥体对象已经存在，此时我们激活已经存在的前一个实例，然后退出当前进程即可。

完整的代码下载地址请[单击这里](https://github.com/baloonwj/mybooksources)。

## 总结
在 Windows 上有太多的应用场景只允许程序启动一个示例，而其原理就是本节所介绍的，希望读者可以理解并掌握该方法。如何保证程序只会启动一个实例是 Windows 开发最常用的应用场景之一，建议读者自己练习一下，尽量掌握。


---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)