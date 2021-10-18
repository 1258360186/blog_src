---
title: 线程 ID 的用途及原理
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
categories:
  - C/C++ 多线程编程精髓
date: 2021-10-18 12:13:20
---
#### 音乐小港
{% meting "1361335791" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# 线程 ID 的用途及原理
前面介绍了如何创建一个线程，本讲我们来介绍一下线程 ID 的内容，线程 ID 一般是用于标识一个线程的整形数值。

## 线程 ID
一个线程创建成功以后，我们可以拿到一个线程 ID。我们可以使用线程 ID 来标识和区分线程，例如在日志文件中，把打印日志的所在的线程 ID 也一起打印出来，我们通过线程 ID 来确定日志内容是不是属于同一个线程上下文。创建线程时，上文也介绍了可以通过 `pthread_create` 函数的第一个参数 `thread` （linux平台）和 `CreateThread` 函数的最后一个参数 `lpThreadId` （Windows平台）得到线程的 ID。大多数时候，我们需要在当前调用线程中获取当前线程的 ID，在 Linux 平台上可以使用 `pthread_self` 函数（还有另外两种方式，下问介绍），在 Windows 平台上可以使用 `GetCurrentThreadID` 函数获取，这两个函数的签名分别如下：
```
pthread_t pthread_self(void);

DWORD GetCurrentThreadId();
```
这两个函数比较简单，这里就不介绍了，无论是 `pthread_t` 还是 DWORD 类型，都是一个 32 位无符号整型值。

在 Windows 7 操作系统中可以在任务管理器中查看某个进程的线程数量：

![multithreaded1](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded1.37epdcvvdt40.jpg)

上图中标红的一栏即每个进程的线程数量，例如对于 vmware-tray.exe 进程一共有三个线程。如果读者打开任务管理器没有看到**线程数**这一列，可以点击任务管理器的 **【查看】**- **【选择列**】菜单，在弹出的对话框中勾选**线程数**即可显示出**线程数**这一列。

## Linux 系统线程 ID 的本质
Linux 系统中有三种方式可以获取一个线程的 ID。

### 方法一

调用 `pthread_create` 函数时，第一个参数在函数调用成功后可以得到线程 ID：
```
#include <pthread.h>

pthread_t tid;
pthread_create(&tid, NULL, thread_proc, NULL);
```
`pthread_create` 函数我们在前面篇幅中已经介绍过了。

### 方法二

在需要获取 ID 的线程中调用 `pthread_self()` 函数获取。
```
#include <pthread.h>

pthread_t tid = pthread_self();
```
### 方法三

通过系统调用获取线程 ID
```
#include <sys/syscall.h>
#include <unistd.h>

int tid = syscall(SYS_gettid);
```
**方法一**和**方法二**获取的线程 ID 结果是一样的，这是一个 `pthread_t`，输出时本质上是一块内存空间地址，示意图如下：

![multithreaded2](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded2.4kg2ddtsxea0.jpg)

由于不同的进程可能有同样地址的内存块，因此**方法一**和**方法二**获取的线程 ID 可能不是全系统唯一的，一般是一个很大的数字（内存地址）。而**方法三**获取的线程 ID 是系统范围内全局唯一的，一般是一个不会太大的整数，这个数字也是就是所谓的 `LWP` （`Light Weight Process`，轻量级进程，早期的 Linux 系统的线程是通过进程来实现的，这种线程被称为轻量级线程）的 ID。

我们来看一段具体的代码：
```
#include <sys/syscall.h>
#include <unistd.h>
#include <stdio.h>
#include <pthread.h>

void* thread_proc(void* arg)
{
    pthread_t* tid1 = (pthread_t*)arg;
    int tid2 = syscall(SYS_gettid);
    pthread_t tid3 = pthread_self();

    while(true)
    {
        printf("tid1: %ld, tid2: %ld, tid3: %ld\n", *tid1, tid2, tid3);
        sleep(1);
    }

}

int main()
{    
    pthread_t tid;
    pthread_create(&tid, NULL, thread_proc, &tid);

    pthread_join(tid, NULL);

    return 0;
}
```
上述代码在新开的线程中使用上面介绍的三种方式获取线程 ID，并打印出来，输出结果如下：
```
tid1: 140185007511296, tid2: 60837, tid3: 140185007511296
```
`tid2` 即 `LWP` 的 ID，而 `tid1` 和 `tid3` 是一个内存地址，转换成 16 进制即：
```
0x7F7F5D935700
```
这与我们用 `pstack` 命令看到的线程 ID 是一样的：
```
[root@localhost ~]# ps -efL | grep linuxtid
root     60712 60363 60712  0    2 13:25 pts/1    00:00:00 ./linuxtid
root     60712 60363 60713  0    2 13:25 pts/1    00:00:00 ./linuxtid
root     60720 60364 60720  0    1 13:25 pts/3    00:00:00 grep --color=auto linuxtid
[root@localhost ~]# pstack 60712
Thread 2 (Thread 0x7fd897a50700 (LWP 60713)):
#0  0x00007fd897b15e2d in nanosleep () from /lib64/libc.so.6
#1  0x00007fd897b15cc4 in sleep () from /lib64/libc.so.6
#2  0x0000000000400746 in thread_proc (arg=0x7fff390921c8) at linuxtid.cpp:15
#3  0x00007fd898644dd5 in start_thread () from /lib64/libpthread.so.0
#4  0x00007fd897b4eead in clone () from /lib64/libc.so.6
Thread 1 (Thread 0x7fd898a6e740 (LWP 60712)):
#0  0x00007fd898645f47 in pthread_join () from /lib64/libpthread.so.0
#1  0x000000000040077e in main () at linuxtid.cpp:25
[root@localhost ~]# ps -ef | grep linuxtid
root     60838 60363  0 14:27 pts/1    00:00:00 ./linuxtid
root     60846 60364  0 14:28 pts/3    00:00:00 grep --color=auto linuxtid
[root@localhost ~]# pstack 60838
Thread 2 (Thread 0x7f7f5d935700 (LWP 60839)):
#0  0x00007f7f5d9fae2d in nanosleep () from /lib64/libc.so.6
#1  0x00007f7f5d9facc4 in sleep () from /lib64/libc.so.6
#2  0x0000000000400746 in thread_proc (arg=0x7fff0523ae68) at linuxtid.cpp:15
#3  0x00007f7f5e529dd5 in start_thread () from /lib64/libpthread.so.0
#4  0x00007f7f5da33ead in clone () from /lib64/libc.so.6
Thread 1 (Thread 0x7f7f5e953740 (LWP 60838)):
#0  0x00007f7f5e52af47 in pthread_join () from /lib64/libpthread.so.0
#1  0x000000000040077e in main () at linuxtid.cpp:25
```
## C++11 的获取当前线程 ID 的方法
`C++11` 的线程库可以使用 `std::thisthread` 类的 `getid` 获取当前线程的 id，这是一个类静态方法。

当然也可以使用 `std::thread` 的 `get_id` 获取指定线程的 id，这是一个类实例方法。

但是 `get_id` 方法返回的是一个包装类型的 `std::thread::id` 对象，不可以直接强转成整型，也没有提供任何转换成整型的接口。所以，我们一般使用 `std::cout` 这样的输出流来输出，或者先转换为 `std::ostringstream` 对象，再转换成字符串类型，然后把字符串类型转换成我们需要的整型。这一点，个人觉得算是 `C++11` 线程库获取线程 id 一个不太方便的地方。
```
//test_cpp11_thread_id.cpp
#include <thread>
#include <iostream>
#include <sstream>

void worker_thread_func()
{
    while (true)
    {

    }
}

int main()
{
    std::thread t(worker_thread_func);
    //获取线程 t 的 ID
    std::thread::id worker_thread_id = t.get_id();
    std::cout << "worker thread id: " << worker_thread_id << std::endl;

    //获取主线程的线程 ID
    std::thread::id main_thread_id = std::this_thread::get_id();
    //先将 std::thread::id 转换成 std::ostringstream 对象
    std::ostringstream oss;
    oss << main_thread_id;
    //再将 std::ostringstream 对象转换成std::string
    std::string str = oss.str();
    std::cout << "main thread id: " << str << std::endl;
    //最后将 std::string 转换成整型值
    unsigned long long threadid = std::stoull(str);
    std::cout << "main thread id: " << threadid << std::endl;

    while (true)
    {
        //权宜之计，让主线程不要提前退出
    }

    return 0;
}
```
在 Linux x64 系统上编译并运行程序，输出结果如下：
```
[root@myaliyun codes]# g++ -g -o test_cpp11_thread_id test_cpp11_thread_id.cpp -lpthread
[root@myaliyun codes]# ./test_cpp11_thread_id 
worker thread id: 139875808245504
main thread id: 139875825641280
main thread id: 139875825641280
```
编译成 Windows x86 程序运行结果如下：

![multithreaded3](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded3.3o0vg5lx5tk0.jpg)

## 总结
线程 ID 在实际编码中是一个很重要的上下文信息，因此熟练地获取某个线程的线程 ID，是多线程编程的基本功之一。


---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)