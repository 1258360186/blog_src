---
title: C++ 11/14/17 线程资源同步对象
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
  - GDB
categories:
  - C/C++ 多线程编程
date: 2021-10-18 14:12:33
---
#### 音乐小港
{% meting "1818178340" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# C++ 11/14/17 线程资源同步对象
在 C/C++ 语言中直接使用操作系统提供的多线程资源同步 API 虽然功能强大，但毕竟存在诸多限制，且同样的代码却不能同时兼容 Windows 和 Linux 两个平台；再者 C/C++ 这种传统语言的使用份额正在被 Java、Python、Go 等语言慢慢蚕食，很大一部分原因是 C/C++ 这门编程语言在一些功能上缺少“完备性”，如对线程同步技术的支持，而这些功能在像 Java、Python、Go 中是标配。

因此，`C++ 11` 标准新加入了很多现代语言标配的东西，其中线程资源同步对象就是其中很重要的一部分。本小节将讨论 `C++ 11` 标准中新增的用于线程同步的 `std::mutex` 和 `std::condition_variable` 对象的用法，有了它们我们就可以写出跨平台的多线程程序了。

## std::mutex 系列
关于 `mutex` 的基本概念上文已经介绍过了，这里不再赘述。

`C++ 11/14/17` 中提供了如下 `mutex` 系列类型：

|互斥量|	版本|	作用|
|---|---|---|
|mutex|	C++11|	最基本的互斥量|
|timed_mutex|	C++11|	有超时机制的互斥量|
|recursive_mutex|	C++11|	可重入的互斥量|
|recursive_timed_mutex|	C++11|	结合 timed_mutex 和 recursive_mutex 特点的互斥量|
|shared_timed_mutex|	C++14|	具有超时机制的可共享互斥量|
|shared_mutex|	C++17|	共享的互斥量|
这个系列的对象均提供了加锁（lock）、尝试加锁（trylock）和解锁（unlock）的方法，我们以 std::mutex 为例来看一段示例代码：
```
#include <iostream>
#include <chrono>
#include <thread>
#include <mutex>

// protected by g_num_mutex
int g_num = 0;  
std::mutex g_num_mutex;

void slow_increment(int id) 
{
    for (int i = 0; i < 3; ++i) {
        g_num_mutex.lock();
        ++g_num;
        std::cout << id << " => " << g_num << std::endl;
        g_num_mutex.unlock();

        //sleep for 1 second
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
}

int main()
{
    std::thread t1(slow_increment, 0);
    std::thread t2(slow_increment, 1);
    t1.join();
    t2.join();

    return 0;
}
```
上述代码中，创建了两个线程 `t1` 和 `t2`，在线程函数的 `for` 循环中调用 `std::mutex.lock()` 和 `std::mutex.unlock()` 对全局变量` g_num` 进行保护。编译程序并输出结果如下：
```
[root@localhost testmultithread]# g++ -g -o mutex c11mutex.cpp -std=c++0x -lpthread
[root@localhost testmultithread]# ./mutex 
0 => 1
1 => 2
0 => 3
1 => 4
1 => 5
0 => 6
```
> 注意：如果你在 Linux 下编译和运行程序，在编译时需要链接 pthread 库，否则能够正常编译但是运行时程序会崩溃，崩溃原因：
>
> terminate called after throwing an instance of 'std::system_error'
>
> what(): Enable multithreading to use std::thread: Operation not permitted

为了避免死锁，`std::mutex.lock()` 和 `std::mutex::unlock() `方法需要成对使用，但是如上文介绍的如果一个函数中有很多出口，而互斥体对象又是需要在整个函数作用域保护的资源，那么在编码时因为忘记在某个出口处调用 `std::mutex.unlock `而造成死锁，上文中推荐使用利用 `RAII` 技术封装这两个接口，其实 `C++ 11` 标准也想到了整个问题，因为已经为我们提供了如下封装：

|互斥量管理|	版本|	作用|
|---|---|---|
|lock_guard|	C++11|	基于作用域的互斥量管理|
|unique_lock|	C++11|	更加灵活的互斥量管理|
|shared_lock|	C++14|	共享互斥量的管理|
|scope_lock|	C++17|	多互斥量避免死锁的管理|
我们这里以 `std::lock_guard` 为例：
```
void func()
{
    std::lock_guard<std::mutex> guard(mymutex);
    //在这里放被保护的资源操作
}
```
`mymutex` 的类型是 `std::mutex`，在 `guard` 对象的构造函数中，会自动调用` mymutex.lock()` 方法加锁，当该函数出了作用域后，调用 `guard` 对象时析构函数时会自动调用 `mymutex.unlock()` 方法解锁。

注意： `mymutex` 生命周期必须长于函数 `func` 的作用域，很多人在初学这个利用 `RAII` 技术封装的 `std::lock_guard` 对象时，可能会写出这样的代码：
```
//错误的写法，这样是没法在多线程调用该函数时保护指定的数据的
void func()
{
    std::mutex m;
    std::lock_guard<std::mutex> guard(m);
    //在这里放被保护的资源操作
}
```
## std::mutex 重复加锁问题
另外，如果一个 `std::mutex` 对象已经调用了 `lock()` 方法，再次调用时，其行为是未定义的，这是一个错误的做法。所谓“行为未定义”即在不同平台上可能会有不同的行为。
```
#include <mutex>

int main()
{
    std::mutex m;
    m.lock();
    m.lock();
    m.unlock();

    return 0;
}
```
实际测试时，上述代码重复调用 `std::mutex.lock()` 方法在 Windows 平台上会引起程序崩溃。如下图所示：

![multithreaded14](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded14.5c12po8n0gk0.jpg)

上述代码在 Linux 系统上运行时会阻塞在第二次调用 `std::mutex.lock()` 处，验证结果如下：
```
[root@localhost testmultithread]# g++ -g -o mutexlock mutexlock.cpp -std=c++0x -lpthread
[root@localhost testmultithread]# gdb mutexlock
Reading symbols from /root/testmultithread/mutexlock...done.
(gdb) r
Starting program: /root/testmultithread/mutexlock 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
^C
Program received signal SIGINT, Interrupt.
0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64 libgcc-4.8.5-36.el7.x86_64 libstdc++-4.8.5-36.el7.x86_64
(gdb) bt
#0  0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
#1  0x00007ffff7bc8dcb in _L_lock_883 () from /lib64/libpthread.so.0
#2  0x00007ffff7bc8c98 in pthread_mutex_lock () from /lib64/libpthread.so.0
#3  0x00000000004006f7 in __gthread_mutex_lock (__mutex=0x7fffffffe3e0)
    at /usr/include/c++/4.8.2/x86_64-redhat-linux/bits/gthr-default.h:748
#4  0x00000000004007a2 in std::mutex::lock (this=0x7fffffffe3e0) at /usr/include/c++/4.8.2/mutex:134
#5  0x0000000000400777 in main () at mutexlock.cpp:7
(gdb) f 5
#5  0x0000000000400777 in main () at mutexlock.cpp:7
7        m.lock();
(gdb) l
2    
3    int main()
4    {
5        std::mutex m;
6        m.lock();
7        m.lock();
8        m.unlock();
9    
10        return 0;
11    }
(gdb)
```
我们使用 `gdb` 运行程序，然后使用 `bt` 命令看到程序确实阻塞在第二个 `m.lock()` 的地方（代码第 7 行）。

不管怎样，对一个已经调用 `lock()` 方法再次调用 `lock()` 方法的做法是错误的，我们实际开发中要避免这么做。

> 有不少开发者诟病 C++ 新标准的多线程库原因之一是 `C++ 11` 引入了 `std::mutex`，却到` C++ 17` 才引入 `std::shared_mutex`，这给使用带来了非常不方便的地方。

## std::condition_variable
`C++ 11` 提供了 `std::condition_variable` 这个类代表条件变量，与 Linux 系统原生的条件变量一样，同时提供了等待条件变量满足的 `wait` 系列方法（`wait`、`wait_for`、`wait_until` 方法），发送条件信号使用 `notify` 方法（`notify_one `和 `notify_all` 方法），当然使用 `std::condition_variable` 对象时需要绑定一个 `std::unique_lock` 或 `std::lock_guard` 对象。

> `C++ 11` 中 `std::condition_variable` 不再需要显式调用方法初始化和销毁。

我们将上文中介绍 Linux 条件变量的例子改写成 `C++ 11` 版本：
```
#include <thread>
#include <mutex>
#include <condition_variable>
#include <list>
#include <iostream>

class Task
{
public:
    Task(int taskID)
    {
        this->taskID = taskID;
    }

    void doTask()
    {
        std::cout << "handle a task, taskID: " << taskID << ", threadID: " << std::this_thread::get_id() << std::endl; 
    }

private:
    int taskID;
};

std::mutex                mymutex;
std::list<Task*>          tasks;
std::condition_variable   mycv;

void* consumer_thread()
{    
    Task* pTask = NULL;
    while (true)
    {
        std::unique_lock<std::mutex> guard(mymutex);
        while (tasks.empty())
        {               
            //如果获得了互斥锁，但是条件不合适的话，pthread_cond_wait会释放锁，不往下执行
            //当发生变化后，条件合适，pthread_cond_wait将直接获得锁
            mycv.wait(guard);
        }

        pTask = tasks.front();
        tasks.pop_front();

        if (pTask == NULL)
            continue;

        pTask->doTask();
        delete pTask;
        pTask = NULL;       
    }

    return NULL;
}

void* producer_thread()
{
    int taskID = 0;
    Task* pTask = NULL;

    while (true)
    {
        pTask = new Task(taskID);

        //使用括号减小guard锁的作用范围
        {
            std::lock_guard<std::mutex> guard(mymutex);
            tasks.push_back(pTask);
            std::cout << "produce a task, taskID: " << taskID << ", threadID: " << std::this_thread::get_id() << std::endl; 
        }

        //释放信号量，通知消费者线程
        mycv.notify_one();

        taskID ++;

        //休眠1秒
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }

    return NULL;
}

int main()
{
    //创建5个消费者线程
    std::thread consumer1(consumer_thread);
    std::thread consumer2(consumer_thread);
    std::thread consumer3(consumer_thread);
    std::thread consumer4(consumer_thread);
    std::thread consumer5(consumer_thread);

    //创建一个生产者线程
    std::thread producer(producer_thread);

    producer.join();
    consumer1.join();
    consumer2.join();
    consumer3.join();
    consumer4.join();
    consumer5.join();

    return 0;
}
```
编译并执行程序输出结果如下所示：
```
[root@localhost testmultithread]# g++ -g -o cpp11cv cpp11cv.cpp -std=c++0x -lpthread
[root@localhost testmultithread]# ./cpp11cv 
produce a task, taskID: 0, threadID: 140427590100736
handle a task, taskID: 0, threadID: 140427623671552
produce a task, taskID: 1, threadID: 140427590100736
handle a task, taskID: 1, threadID: 140427632064256
produce a task, taskID: 2, threadID: 140427590100736
handle a task, taskID: 2, threadID: 140427615278848
produce a task, taskID: 3, threadID: 140427590100736
handle a task, taskID: 3, threadID: 140427606886144
produce a task, taskID: 4, threadID: 140427590100736
handle a task, taskID: 4, threadID: 140427598493440
produce a task, taskID: 5, threadID: 140427590100736
handle a task, taskID: 5, threadID: 140427623671552
produce a task, taskID: 6, threadID: 140427590100736
handle a task, taskID: 6, threadID: 140427632064256
produce a task, taskID: 7, threadID: 140427590100736
handle a task, taskID: 7, threadID: 140427615278848
produce a task, taskID: 8, threadID: 140427590100736
handle a task, taskID: 8, threadID: 140427606886144
produce a task, taskID: 9, threadID: 140427590100736
handle a task, taskID: 9, threadID: 140427598493440
...更多输出结果省略...
```
> 需要注意的是，如果在 Linux 平台上，`std::condition_variable` 也存在虚假唤醒这一现象，如何避免与上文中介绍 Linux 原生的条件变量方法一样。

## 总结
除了 `std::mutex`、`std::condition_variable` 类，`C++ 11/14/17` 还同步引入的其他一些多线程资源同步辅助类，如 `std::lock_guard`、`std::unique_lock` 等，它们被加入 C++ 语言中极大地方便了 C++ 的跨平台开发。当然，读者一定要明白，这些引入的对象其实就是前面章节介绍的 `mutex`、条件变量等操作系统平台的多线程资源同步 API 的封装。

> 从笔者自身的开发经历来说，自从有了 `std::mutex`、`std::condition_variable` 等对象，我在项目中大量使用他们，很少再使用操作系统本身提供的多线程资源同步 API 了，它们也在各种开源 C++ 项目中广泛使用。


---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)