---
title: Linux 线程同步之条件变量
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
  - GDB
categories:
  - C/C++ 多线程编程
date: 2021-10-18 13:56:20
---
#### 音乐小港
{% meting "2059056" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Linux 线程同步之条件变量
有人说 Linux 条件变量（`Condition Variable`）是最不会用错的一种线程同步对象，确实是这样，但这必须建立在你对条件变量熟练使用的基础之上。我们先来讨论一下为什么会存在条件变量这样一种机制。

## 为什么需要使用条件变量
实际应用中，我们常常会有类似如下需求：
```
//以下是伪码，m 的类型是 pthread_mutex_t，并且已经初始化过了
int WaitForTrue()
{
    pthread_mutex_lock(&m);
    while (condition is false)      //条件不满足
    {
        pthread_mutex_unlock(&m);   //解锁等待其他线程改变 condition
        sleep(n);                   //睡眠n秒
        //n秒后再次加锁验证条件是否满足
        pthread_mutex_lock(&m);
    }

    return 1;
}
```
以上逻辑可以表示成如下流程图：

![multithreaded13](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20210918/multithreaded13.1lhc2y26j7mo.jpg)

这段逻辑的用途是我们需要反复判断一个多线程共享条件是否满足，一直到该条件满足为止，由于该条件被多个线程操作因此每次判断之前都需要进行加锁操作，判断完毕后需要进行解锁操作。但是上述逻辑存在严重的效率问题，假设解锁离开临界区后，此时由于其他线程修改了条件导致条件满足了，此时程序仍然需要睡眠 n 秒后才能得到反馈。因此我们需要这样一种机制：

> 某个线程 A 在条件不满足的情况下，主动让出互斥量，让其他线程去折腾，线程在此处等待，等待条件的满足；一旦条件满足，线程就可以被立刻唤醒。线程 A 之所以可以安心等待，依赖的是其他线程的协作，它确信会有一个线程在发现条件满足以后，将向它发送信号，并且让出互斥量。如果其他线程不配合（不发信号，不让出互斥量），这个主动让出互斥量并等待事件发生的线程 A 就真的要等到花儿都谢了。

这个例子解释了为什么需要条件等待，但是条件等待还不是条件变量的全部功能。

## 条件变量为什么要与互斥体对象结合
很多第一次学习 Linux 条件变量的读者会觉得困惑：为什么条件变量一定要与一个互斥体对象结合使用？我们来看下，假设条件变量不与互斥体对象结合的效果。
```
1 //m的类型是 pthread_mutex_t，并且已经初始化过了，cv 是条件变量
2 pthread_mutex_lock(&m)
3 while(condition_is_false)
4 {
5     pthread_mutex_unlock(&m);
6     //解锁之后，等待之前，可能条件已经满足，信号已经发出，但是该信号可能会被错过
7     cond_wait(&cv);
8     pthread_mutex_lock(&m);
9 }
```
上述代码中，假设线程 A 执行完第 5 行代码 `pthread_mutex_unlock(&m)`; 后 CPU 时间片被剥夺，此时另外一个线程 B 获得该互斥体对象 m，然后发送条件信号，等线程 A 重新获得时间片后，由于该信号已经被错过了，这样可能会导致线程 A 在 第 7 行 `cond_wait(&cv)`; 无限阻塞下去。

造成这个问题的根源是释放互斥体对象与条件变量等待唤醒不是原子操作，即解锁和等待这两个步骤必须是同一个原子性的操作，以确保 `cond_wait` 唤醒之前不会有其他线程获得这个互斥体对象。

## 条件变量的使用
介绍了这么多，我们来正式介绍一下条件变量相关的系统 API 的使用方法。

条件变量的初始化和销毁可以使用如下 API 函数：
```
int pthread_cond_init(pthread_cond_t* cond, const pthread_condattr_t* attr);
int pthread_cond_destroy(pthread_cond_t* cond);
```
在 Linux 系统中 `pthread_cond_t` 即是条件变量的类型，当然和前面介绍的互斥体一样，也可以使用如下方式去初始化一个条件变量：
```
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
```
等待条件变量的满足可以使用如下 API 函数：
```
int pthread_cond_wait(pthread_cond_t* restrict cond, pthread_mutex_t* restrict mutex);
int pthread_cond_timedwait(pthread_cond_t* restrict cond, pthread_mutex_t* restrict mutex, const struct timespec* restrict abstime);
```
一般情况下如果条件变量代表的条件不会满足，调用 `pthread_cond_wait` 的线程会一直等待下去；`pthread_cond_timedwait` 是 `pthread_cond_wait` 非阻塞版本，它会在指定时间内等待条件满足，超过参数 `abstime` 设置的时候后 `pthread_cond_timedwait` 函数会立即返回。

> 注意：对于参数 `abstime`，正如其名字暗示的，这是一个 `absolute time`（绝对时间），也就是说，如果你打算让函数等待 5 秒，那么你应该先得到当前系统的时间，然后加上 5 秒计算出最终的时间作为参数 `abstime` 的值。

因调用 `pthread_cond_wait` 等待的线程可以被以下 API 函数唤醒：
```
int pthread_cond_signal(pthread_cond_t* cond);
int pthread_cond_broadcast(pthread_cond_t* cond);     
```
`pthread_cond_signal` 一次唤醒一个线程，如果有多个线程调用 `pthread_cond_wait` 等待，具体哪个线程被唤醒是不确定的（可以认为是随机的）；`pthread_cond_broadcast` 可以同时唤醒多个调用 `pthread_cond_wait` 等待的线程。前者相当于发送一次条件通知，后者广播一次条件通知。成功等待到条件信号，`pthread_cond_signal` 和 `pthread_cond_broadcast` 返回 0，反之返回非 0 值，具体错误原因可以通过错误码 `errno` 获得。

我们将前文中介绍信号量的示例代码用条件变量来改写下：
```
#include <pthread.h>
#include <errno.h>
#include <unistd.h>
#include <list>
#include <semaphore.h>
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
        std::cout << "handle a task, taskID: " << taskID << ", threadID: " << pthread_self() << std::endl; 
    }

private:
    int taskID;
};

pthread_mutex_t  mymutex;
std::list<Task*> tasks;
pthread_cond_t   mycv;

void* consumer_thread(void* param)
{    
    Task* pTask = NULL;
    while (true)
    {
        pthread_mutex_lock(&mymutex);
        while (tasks.empty())
        {               
            //如果获得了互斥锁，但是条件不合适的话，pthread_cond_wait 会释放锁，不往下执行
            //当发生变化后，条件合适，pthread_cond_wait 将直接获得锁
            pthread_cond_wait(&mycv, &mymutex);
        }

        pTask = tasks.front();
        tasks.pop_front();

        pthread_mutex_unlock(&mymutex);

        if (pTask == NULL)
            continue;

        pTask->doTask();
        delete pTask;
        pTask = NULL;       
    }

    return NULL;
}

void* producer_thread(void* param)
{
    int taskID = 0;
    Task* pTask = NULL;

    while (true)
    {
        pTask = new Task(taskID);

        pthread_mutex_lock(&mymutex);
        tasks.push_back(pTask);
        std::cout << "produce a task, taskID: " << taskID << ", threadID: " << pthread_self() << std::endl; 

        pthread_mutex_unlock(&mymutex);

        //释放信号量，通知消费者线程
        pthread_cond_signal(&mycv);

        taskID ++;

        //休眠1秒
        sleep(1);
    }

    return NULL;
}

int main()
{
    pthread_mutex_init(&mymutex, NULL);
    pthread_cond_init(&mycv, NULL);

    //创建 5 个消费者线程
    pthread_t consumerThreadID[5];
    for (int i = 0; i < 5; ++i)
    {
        pthread_create(&consumerThreadID[i], NULL, consumer_thread, NULL);
    }

    //创建一个生产者线程
    pthread_t producerThreadID;
    pthread_create(&producerThreadID, NULL, producer_thread, NULL);

    pthread_join(producerThreadID, NULL);

    for (int i = 0; i < 5; ++i)
    {
        pthread_join(consumerThreadID[i], NULL);
    }

    pthread_cond_destroy(&mycv);
    pthread_mutex_destroy(&mymutex);

    return 0;
}
```
编译并执行上述程序，输出结果如下：
```
[root@localhost testsemaphore]# g++ -g -o cv cv.cpp -lpthread
[root@localhost testsemaphore]# ./cv
produce a task, taskID: 0, threadID: 140571200554752
handle a task, taskID: 0, threadID: 140571242518272
produce a task, taskID: 1, threadID: 140571200554752
handle a task, taskID: 1, threadID: 140571225732864
produce a task, taskID: 2, threadID: 140571200554752
handle a task, taskID: 2, threadID: 140571208947456
produce a task, taskID: 3, threadID: 140571200554752
handle a task, taskID: 3, threadID: 140571242518272
produce a task, taskID: 4, threadID: 140571200554752
handle a task, taskID: 4, threadID: 140571234125568
produce a task, taskID: 5, threadID: 140571200554752
handle a task, taskID: 5, threadID: 140571217340160
produce a task, taskID: 6, threadID: 140571200554752
handle a task, taskID: 6, threadID: 140571225732864
produce a task, taskID: 7, threadID: 140571200554752
handle a task, taskID: 7, threadID: 140571208947456
produce a task, taskID: 8, threadID: 140571200554752
handle a task, taskID: 8, threadID: 140571242518272
...更多输出结果省略...
```
条件变量最关键的一个地方就是需要清楚地记得 `pthread_cond_wait` 在条件满足与不满足时的两种行为，这是难点也是**重点**：

- 当 `pthread_cond_wait` 函数阻塞时，它会释放其绑定的互斥体，并阻塞线程，因此在调用该函数前应该对互斥体有个加锁操作（上述代码的第 34 行的 `pthread_mutex_lock(&mymutex)`;）。
- 当收到条件信号时， `pthread_cond_wait` 会返回并对其绑定的互斥体进行加锁，因此在其下面一定有个对互斥体进行解锁的操作（上述代码的第 45 行 `pthread_mutex_unlock(&mymutex)`;）。
## 条件变量的虚假唤醒
上面将互斥量和条件变量配合使用的示例代码中有个很有意思的地方，就是用了 `while` 语句，醒来 之后要再次判断条件是否满足。
```
while (tasks.empty())
{                
    pthread_cond_wait(&mycv, &mymutex);
}
```
为什么不写成：
```
if (tasks.empty())
{                
    pthread_cond_wait(&mycv, &mymutex);
}
```
答案是不得不如此。因为可能某次操作系统唤醒 `pthread_cond_wait` 时 `tasks.empty()` 可能仍然为 `true`，言下之意就是操作系统可能会在一些情况下唤醒条件变量，即使没有其他线程向条件变量发送信号，等待此条件变量的线程也有可能会醒来。我们将条件变量的这种行为称之为 **虚假唤醒** （spurious wakeup）。因此将条件（判断 `tasks.empty()` 为 `true`）放在一个 `while` 循环中意味着光唤醒条件变量不行，还必须条件满足程序才能继续执行正常的逻辑。

这看起来这像是个 `bug`，但它在 Linux 系统中是实实在在存在的。为什么会存在虚假唤醒呢？一个原因是：`pthread_cond_wait` 是 `futex` 系统调用，属于阻塞型的系统调用，当系统调用被信号中断的时候，会返回 ﹣1，并且把 `errno` 错误码置为 `EINTR`。很多这种系统调用为了防止被信号中断都会重启系统调用（即再次调用一次这个函数），代码如下：
```
pid_t r_wait(int *stat_loc)
{
    int retval;
    //wait 函数因为被信号中断导致调用失败会返回 ﹣1，错误码是 EINTR  
    //注意：这里的 while 循环体是一条空语句
    while(((retval = wait(stat_loc)) == -1 && (errno == EINTR));

    return retval;
}
```
但是 `pthread_cond_wait` 用途有点不一样，假设 `pthread_cond_wait` 函数被信号中断了，在 `pthread_cond_wait` 返回之后，到重新调用之前，`pthread_cond_signal` 或 `pthread_cond_broadcast` 可能已经调用过。一旦错失，可能由于条件信号不再产生，再次调用 `pthread_cond_wait` 将导致程序无限制地等待下去。为了避免这种情况，宁可虚假唤醒，也不能再次调用 `pthread_cond_wait`，以免陷入无穷的等待中。

除了上面的信号因素外，还存在以下情况：条件满足了发送信号，但等到调用 `pthread_cond_wait` 的线程得到 CPU 资源时，条件又再次不满足了。

好在无论是哪种情况，醒来之后再次测试条件是否满足就可以解决虚假等待的问题。这就是使用 `while` 循环来判断条件，而不是使用 `if` 语句的原因。

## 条件变量信号丢失问题
上文中，我们介绍了，如果一个条件变量信号条件产生时（调用 `pthread_cond_signal` 或 `pthread_cond_broadcast`），没有相关的线程调用 `pthread_cond_wait` 捕获该信号，那么该信号条件就会永久性地丢失了，再次调用 `pthread_cond_wait` 会导致永久性的阻塞。这种情况在设计那些条件变量信号条件只会产生一次的逻辑中尤其需要注意，例如假设现在某个程序有一批等待条件变量的线程，和一个只产生一次条件变量信号的线程。为了让你的等待条件变量的线程能正常运行不阻塞，你的逻辑中，一定要确保等待的线程在产生条件变量信号的线程发送条件信号之前调用 `pthread_cond_wait` 。

> 这和生活中的很多例子一样，即许多事情你只有一次机会，必须提前准备好再去尝试这次机会，这个机会不会等待你的准备，一旦错过，就不会再有第二次机会了。

## 总结
本节介绍了学习 Linux 条件变量需要掌握的重难点知识，条件变量是最常用的一种多线程编程同步技术之一，也是面试高频问题之一，建议打算从事相关工作的读者务必理解和熟练使用它。



---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)