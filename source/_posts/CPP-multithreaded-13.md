---
title: Linux 线程同步之读写锁
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
  - GDB
categories:
  - C/C++ 多线程编程
date: 2021-10-18 14:04:23
---
#### 音乐小港
{% meting "1826142553" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Linux 线程同步之读写锁
在实际应用中，很多时候对共享变量的访问有以下特点：

> 大多数情况下线程只是读取共享变量的值，并不修改，只有在极少数情况下，线程才会真正地修改共享变量的值。

对于这种情况，读请求之间是无需同步的，它们之间的并发访问是安全的。然而写请求必须锁住读请求和其他写请求。

这种情况在实际中是存在的，如读取一个全局对象的状态属性，大多数情况下这个状态属性值是不会变化的，偶尔才会出现被修改的情况。如果使用互斥量，完全阻止读请求并发，则会造成性能的损失。

## 读写锁使用方法
读写锁在 Linux 系统中使用类型 `pthread_rwlock_t` 表示，读写锁的初始化和销毁使用如下系统 API 函数：
```
#include <pthread.h>

int pthread_rwlock_init(pthread_rwlock_t* rwlock, const pthread_rwlockattr_t* attr);
int pthread_rwlock_destroy(pthread_rwlock_t* rwlock);
```
参数 `rwlock` 即需要初始化和销毁的读写锁对象的地址，参数 `attr` 用于设置读写锁的属性，一般设置未 `NULL` 表示使用默认属性。函数调用成功返回 0，调用失败返回非 0 值，你可以通过检测错误码 `errno` 获取错误原因。

当然，如果你不需要动态创建或者设置非默认属性的读写锁对象，也可以使用如下语法初始化一个读写锁对象：
```
pthread_rwlock_t myrwlock = PTHREAD_RWLOCK_INITIALIZER;
```
下面是三个请求读锁的系统 API 接口：
```
int pthread_rwlock_rdlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_timedrdlock(pthread_rwlock_t* rwlock, const struct timespec* abstime);
```
而下面三个请求写锁的系统 API 接口：
```
int pthread_rwlock_wrlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_timedwrlock(pthread_rwlock_t* rwlock, const struct timespec* abstime);
```
读锁用于共享模式：

- 如果当前读写锁已经被某线程以读模式占有了，其他线程调用 `pthread_rwlock_rdlock` （请求读锁）会立刻获得读锁；
- 如果当前读写锁已经被某线程以读模式占有了，其他线程调用 `pthread_rwlock_wrlock` （请求写锁）会陷入阻塞。

写锁用的是独占模式：

- 如果当前**读写锁**被某线程以**写模式**占有，无论调用 `pthread_rwlock_rdlock` 还是 `pthread_rwlock_wrlock` 都会陷入**阻塞**，即**写模式**下不允许任何**读锁**请求通过，也不允许任何**写锁**请求通过，**读锁**请求和**写锁**请求都要陷入**阻塞**，直到线程释放**写锁**。
可以将上述读写锁逻辑总结成如下表格：

|锁当前状态/其他线程请求锁类型|	请求读锁|	请求写锁|
|---|---|
|无锁|	通过|	通过|
|已经获得读锁|	通过|	阻止|
|已经获得写锁|	阻止|	阻止|
无论是读锁还是写锁，锁的释放都是一个接口：
```
int pthread_rwlock_unlock (pthread_rwlock_t* rwlock);
```
无论是请求读锁还是写锁，都提供了 `trylock` 的功能（`pthread_rwlock_tryrdlock` 和 `pthread_rwlock_trywrlock`），调用线程不会阻塞，而会立即返回。如果能成功获得读锁或者写锁，函数返回 0，如果不能获得读锁或写锁时，函数返回非 0 值，此时错误码 `errno` 是 `EBUSY`。

当然，无论是请求读锁还是写锁都提供了限时等待功能，如果不能获取读写锁，则会陷入阻塞，最多等待到参数 `abstime` 设置的时间；如果仍然无法获得锁，则返回，错误码 `errno` 是 `ETIMEOUT`。

## 读写锁的属性
上文介绍 `pthread_rwlock_init` 函数时，提到其第二个参数可以设置读写锁的属性，读写锁的属性类型是 `pthread_rwlockattr_t` ，`glibc` 引入了如下接口来查询和改变读写锁的类型：
```
#include <pthread.h>

int pthread_rwlockattr_setkind_np(pthread_rwlockattr_t* attr, int pref);
int pthread_rwlockattr_getkind_np(const pthread_rwlockattr_t* attr, int* pref);
```
`pthread_rwlockattr_setkind_np` 的第二个参数 `pref` 即设置读写锁的类型，其取值有如下几种：
```
enum
{
    //读者优先（即同时请求读锁和写锁时，请求读锁的线程优先获得锁）
    PTHREAD_RWLOCK_PREFER_READER_NP, 
    //不要被名字所迷惑，也是读者优先
    PTHREAD_RWLOCK_PREFER_WRITER_NP, 
    //写者优先（即同时请求读锁和写锁时，请求写锁的线程优先获得锁）
    PTHREAD_RWLOCK_PREFER_WRITER_NONRECURSIVE_NP,                 
    PTHREAD_RWLOCK_DEFAULT_NP = PTHREAD_RWLOCK_PREFER_READER_NP
};
```
当然，为了得到一个有效的 `pthread_rwlockattr_t` 对象，你需要调用 `pthread_rwlockattr_init` 函数初始化这样一个属性对象，在你不需要的时候记得使用 `pthread_rwlockattr_destroy` 销毁之：
```
int pthread_rwlockattr_init(pthread_rwlockattr_t* attr);
int pthread_rwlockattr_destroy(pthread_rwlockattr_t* attr);
```
以下代码片段演示了如何初始化一个写者优先的读写锁：
```
pthread_rwlockattr_t attr;
pthread_rwlockattr_init(&attr);
pthread_rwlockattr_setkind_np(&attr, PTHREAD_RWLOCK_PREFER_WRITER_NONRECURSIVE_NP);
pthread_rwlock_t rwlock;
pthread_rwlock_init(&rwlock, &attr);
```
读写锁使用示例
```
#include <pthread.h>
#include <unistd.h>
#include <iostream>

int resourceID = 0;
pthread_rwlock_t myrwlock;

void* read_thread(void* param)
{    
    while (true)
    {
        //请求读锁
        pthread_rwlock_rdlock(&myrwlock);

        std::cout << "read thread ID: " << pthread_self() << ", resourceID: " << resourceID << std::endl;

        //使用睡眠模拟读线程读的过程消耗了很久的时间
        sleep(1);

        pthread_rwlock_unlock(&myrwlock);
    }

    return NULL;
}

void* write_thread(void* param)
{
    while (true)
    {
        //请求写锁
        pthread_rwlock_wrlock(&myrwlock);

        ++resourceID;
        std::cout << "write thread ID: " << pthread_self() << ", resourceID: " << resourceID << std::endl;

        //使用睡眠模拟读线程读的过程消耗了很久的时间
        sleep(1);

        pthread_rwlock_unlock(&myrwlock);
    }

    return NULL;
}

int main()
{
    pthread_rwlock_init(&myrwlock, NULL);

    //创建5个请求读锁线程
    pthread_t readThreadID[5];
    for (int i = 0; i < 5; ++i)
    {
        pthread_create(&readThreadID[i], NULL, read_thread, NULL);
    }

    //创建一个请求写锁线程
    pthread_t writeThreadID;
    pthread_create(&writeThreadID, NULL, write_thread, NULL);

    pthread_join(writeThreadID, NULL);

    for (int i = 0; i < 5; ++i)
    {
        pthread_join(readThreadID[i], NULL);
    }

    pthread_rwlock_destroy(&myrwlock);

    return 0;
}
```
上述程序中创建五个请求读锁的“读”线程和一个请求写锁的“写”线程，共享的资源是一个整形变量 `resourceID`，我们编译并执行得到输出结果：
```
[root@localhost testmultithread]# g++ -g -o rwlock rwlock.cpp -lpthread
[root@localhost testmultithread]# ./rwlock
read thread ID: 140575861593856, resourceID: 0
read thread ID: 140575878379264, resourceID: 0
read thread ID: 140575853201152, resourceID: 0
read thread ID: 140575869986560, resourceID: 0
read thread ID: 140575886771968, resourceID: 0
read thread ID: read thread ID: read thread ID: read thread ID: 140575861593856140575886771968, resourceID: 0, resourceID: 
0
140575878379264read thread ID: 140575869986560, resourceID: 0
, resourceID: 0
140575853201152, resourceID: 0
read thread ID: read thread ID: read thread ID: 140575861593856140575853201152140575886771968, resourceID: , resourceID: 0, resourceID: 00


read thread ID: 140575869986560, resourceID: 0
...更多输出结果省略...
```
上述输出结果，我们验证了两个结论：

- 由于读写锁对象 `myrwlock` 使用了默认属性，其行为是请求读锁的线程优先获得到锁，请求写锁的线程 `write_thread` 很难获得锁的机会，因此结果中基本没有请求写锁线程的输出结果；
- 由于多个请求读锁的线程 `read_thread` 可以自由获得读锁，且代码 15 行（`std::cout << "read thread ID: " << pthread_self() << ", resourceID: " << resourceID << std::endl;`）的输出不是原子性的，因而多个“读”线程的输出可能会交替，出现“错乱”现象。
我们将读写锁对象 `myrwlock` 的属性修改成请求写锁优先，再来试一试：
```
#include <pthread.h>
#include <unistd.h>
#include <iostream>

int resourceID = 0;
pthread_rwlock_t myrwlock;

void* read_thread(void* param)
{    
    while (true)
    {
        //请求读锁
        pthread_rwlock_rdlock(&myrwlock);

        std::cout << "read thread ID: " << pthread_self() << ", resourceID: " << resourceID << std::endl;

        //使用睡眠模拟读线程读的过程消耗了很久的时间
        sleep(1);

        pthread_rwlock_unlock(&myrwlock);
    }

    return NULL;
}

void* write_thread(void* param)
{
    while (true)
    {
        //请求写锁
        pthread_rwlock_wrlock(&myrwlock);

        ++resourceID;
        std::cout << "write thread ID: " << pthread_self() << ", resourceID: " << resourceID << std::endl;

        //使用睡眠模拟读线程读的过程消耗了很久的时间
        sleep(1);

        pthread_rwlock_unlock(&myrwlock);
    }

    return NULL;
}

int main()
{
    pthread_rwlockattr_t attr;
    pthread_rwlockattr_init(&attr);
    //设置成请求写锁优先
    pthread_rwlockattr_setkind_np(&attr, PTHREAD_RWLOCK_PREFER_WRITER_NONRECURSIVE_NP);
    pthread_rwlock_init(&myrwlock, &attr);

    //创建5个请求读锁线程
    pthread_t readThreadID[5];
    for (int i = 0; i < 5; ++i)
    {
        pthread_create(&readThreadID[i], NULL, read_thread, NULL);
    }

    //创建一个请求写锁线程
    pthread_t writeThreadID;
    pthread_create(&writeThreadID, NULL, write_thread, NULL);

    pthread_join(writeThreadID, NULL);

    for (int i = 0; i < 5; ++i)
    {
        pthread_join(readThreadID[i], NULL);
    }

    pthread_rwlock_destroy(&myrwlock);

    return 0;
}
```
编译程序并运行，输出结果如下：
```
[root@localhost testmultithread]# g++ -g -o rwlock2 rwlock2.cpp -lpthread
[root@localhost testmultithread]# ./rwlock2
read thread ID: 140122217539328, resourceID: 0
read thread ID: 140122242717440, resourceID: 0
read thread ID: 140122209146624, resourceID: 0
write thread ID: 140122200753920, resourceID: 1
read thread ID: 140122234324736, resourceID: 1
write thread ID: 140122200753920, resourceID: 2
write thread ID: 140122200753920, resourceID: 3
write thread ID: 140122200753920, resourceID: 4
write thread ID: 140122200753920, resourceID: 5
write thread ID: 140122200753920, resourceID: 6
write thread ID: 140122200753920, resourceID: 7
write thread ID: 140122200753920, resourceID: 8
write thread ID: 140122200753920, resourceID: 9
write thread ID: 140122200753920, resourceID: 10
write thread ID: 140122200753920, resourceID: 11
write thread ID: 140122200753920, resourceID: 12
write thread ID: 140122200753920, resourceID: 13
read thread ID: 140122217539328, resourceID: 13
write thread ID: 140122200753920, resourceID: 14
write thread ID: 140122200753920, resourceID: 15
write thread ID: 140122200753920, resourceID: 16
write thread ID: 140122200753920, resourceID: 17
write thread ID: 140122200753920, resourceID: 18
write thread ID: 140122200753920, resourceID: 19
write thread ID: 140122200753920, resourceID: 20
write thread ID: 140122200753920, resourceID: 21
write thread ID: 140122200753920, resourceID: 22
write thread ID: 140122200753920, resourceID: 23
...更多输出结果省略...
```
由于将 `myrwlock` 设置成请求写锁优先，上述结果中几乎都是 `write_thread` 的输出结果。

我们将 `write_thread` 中的 37 行 `sleep` 语句挪到 39 行后面，增加请求写锁线程的睡眠时间，再看看执行结果。
```
#include <pthread.h>
#include <unistd.h>
#include <iostream>

int resourceID = 0;
pthread_rwlock_t myrwlock;

void* read_thread(void* param)
{    
    while (true)
    {
        //请求读锁
        pthread_rwlock_rdlock(&myrwlock);

        std::cout << "read thread ID: " << pthread_self() << ", resourceID: " << resourceID << std::endl;

        //使用睡眠模拟读线程读的过程消耗了很久的时间
        sleep(1);

        pthread_rwlock_unlock(&myrwlock);
    }

    return NULL;
}

void* write_thread(void* param)
{
    while (true)
    {
        //请求写锁
        pthread_rwlock_wrlock(&myrwlock);

        ++resourceID;
        std::cout << "write thread ID: " << pthread_self() << ", resourceID: " << resourceID << std::endl;

        pthread_rwlock_unlock(&myrwlock);

        //放在这里增加请求读锁线程获得锁的几率
        sleep(1);
    }

    return NULL;
}

int main()
{
    pthread_rwlockattr_t attr;
    pthread_rwlockattr_init(&attr);
    //设置成请求写锁优先
    pthread_rwlockattr_setkind_np(&attr, PTHREAD_RWLOCK_PREFER_WRITER_NONRECURSIVE_NP);
    pthread_rwlock_init(&myrwlock, &attr);

    //创建5个请求读锁线程
    pthread_t readThreadID[5];
    for (int i = 0; i < 5; ++i)
    {
        pthread_create(&readThreadID[i], NULL, read_thread, NULL);
    }

    //创建一个请求写锁线程
    pthread_t writeThreadID;
    pthread_create(&writeThreadID, NULL, write_thread, NULL);

    pthread_join(writeThreadID, NULL);

    for (int i = 0; i < 5; ++i)
    {
        pthread_join(readThreadID[i], NULL);
    }

    pthread_rwlock_destroy(&myrwlock);

    return 0;
}
```
再次编译程序并执行，得到输出结果：
```
[root@localhost testmultithread]# g++ -g -o rwlock3 rwlock3.cpp -lpthread
[root@localhost testmultithread]# ./rwlock3
read thread ID: 140315524790016, resourceID: 0
read thread ID: 140315549968128, resourceID: 0
read thread ID: 140315541575424, resourceID: 0
write thread ID: 140315508004608, resourceID: 1
read thread ID: 140315549968128, resourceID: 1
read thread ID: 140315541575424, resourceID: 1
read thread ID: 140315524790016, resourceID: 1
read thread ID: 140315516397312, resourceID: 1
read thread ID: 140315533182720, resourceID: 1
write thread ID: 140315508004608, resourceID: 2
read thread ID: 140315541575424, resourceID: 2
read thread ID: 140315524790016, resourceID: 2
read thread ID: 140315533182720, resourceID: 2
read thread ID: 140315516397312, resourceID: 2
read thread ID: 140315549968128, resourceID: 2
read thread ID: 140315516397312, resourceID: 2
write thread ID: 140315508004608, resourceID: 3
read thread ID: 140315549968128, resourceID: 3
read thread ID: 140315541575424, resourceID: 3
read thread ID: 140315533182720, resourceID: 3read thread ID: read thread ID: 140315524790016, resourceID: 3
140315516397312, resourceID: 3

read thread ID: read thread ID: read thread ID: 140315524790016140315549968128, resourceID: , resourceID: 33
140315516397312, resourceID: 3
read thread ID: 140315541575424, resourceID: read thread ID: 140315533182720, resourceID: 3
3

write thread ID: 140315508004608, resourceID: 4
read thread ID: 140315516397312, resourceID: 4
read thread ID: 140315541575424, resourceID: 4
read thread ID: 140315524790016, resourceID: 4
read thread ID: 140315549968128, resourceID: 4
read thread ID: 140315533182720, resourceID: 4
read thread ID: 140315524790016, resourceID: 4
read thread ID: 140315541575424, resourceID: 4
write thread ID: 140315508004608, resourceID: 5
read thread ID: 140315516397312, resourceID: 5
read thread ID: 140315541575424, resourceID: 5
read thread ID: 140315524790016, resourceID: 5
read thread ID: 140315533182720, resourceID: 5
read thread ID: 140315549968128, resourceID: 5
```
这次请求读锁的线程和请求写锁的线程的输出结果分布就比较均匀了。

以上例子比较简单，建议读者实际运行一下代码实验一下。

## 总结
`mutex` 多线程之间，无论线程对共享资源是读还是写一概加上锁，加锁期间，不允许其他线程进行任何操作，而读写锁允许多个线程的读操作，因此相对于 `mutex` 提高了效率， 这也是 `boost::mutex` 和 `boost::shared_mutex` 在 Linux 平台的实现原理，前者使用 `mutex` 实现，后者使用读写锁实现。

---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)