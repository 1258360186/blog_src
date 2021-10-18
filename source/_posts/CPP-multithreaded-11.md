---
title: Linux 线程同步之信号量
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
categories:
  - C/C++ 多线程编程精髓
date: 2021-10-18 13:51:03
---
#### 音乐小港
{% meting "1320776019" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Linux 线程同步之信号量
与 Windows 的 Semaphore 对象使用原理一样，Linux 的信号量本质上也是暗含着“资源有多份，可以同时被多个线程访问”的意味，故信号量的原理这里不再赘述。

Linux 信号量常用的一组 API 函数是：
```
#include <semaphore.h>
int sem_init(sem_t* sem, int pshared, unsigned int value);
int sem_destroy(sem_t* sem);
int sem_post(sem_t* sem);
int sem_wait(sem_t* sem);
int sem_trywait(sem_t* sem);
int sem_timedwait(sem_t* sem, const struct timespec* abs_timeout);
```
- 函数 `sem_init` 用于初始化一个信号量，第一个参数 `sem` 传入需要初始化的信号量对象的地址；第二个参数 `pshared` 表示该信号量是否可以被共享，取值为 0 表示该信号量可以在同一个进程多个线程之间共享，取值为非 0 表示可以在多个进程之间共享；第三个参数 `value` 用于设置信号量初始状态下资源的数量。函数 sem_init 函数调用成功返回 0，失败返回 -1，实际编码中只要我们的写法得当一般不用关心该函数的返回值。
- 函数 `sem_destroy` 用于销毁一个信号量。
- 函数 `sem_post` 将信号量的资源计数递增 1，并解锁该信号量对象，这样其他由于使用 `sem_wait` 被阻塞的线程会被唤醒。
- 如果当前信号量资源计数为 0，函数 `sem_wait` 会阻塞调用线程；直到信号量对象的资源计数大于 0 时被唤醒，唤醒后将资源计数递减 1，然后立即返回；函数 `sem_trywait` 是函数 `sem_wait` 的非阻塞版本，如果当前信号量对象的资源计数等于 0，`sem_trywait` 会立即返回不会阻塞调用线程，返回值是 ﹣1，错误码 `errno` 被设置成 `EAGAIN`；函数 `sem_timedwait` 是带有等待时间的版本，等待时间在第二个参数 `abs_timeout` 中设置，这是个结构体的定义如下：
```
struct timespec
{
    time_t tv_sec;      /* 秒 */
    long   tv_nsec;     /* 纳秒 [0 .. 999999999] */
};
```
`sem_timedwait` 在参数 `abs_timeout` 设置的时间内等待信号量对象的资源计数大于0，否则超时返回，返回值为 ﹣1，错误码 `errno` 是 `ETIMEDOUT`。当使用 `sem_timedwait` 时，参数 `abs_timeout` 不能设置为 `NULL`，否则程序会在运行时调用 `sem_timedwait` 产生崩溃。

> 注意：
>
> `sem_wait`、`sem_trywait`、`sem_timedwait` 函数将资源计数递减一时会同时锁定信号量对象，因此当资源计数为 1 时，如果有多个线程调用 `sem_wait` 等函数等待该信号量时，只会有一个线程被唤醒。当 `sem_wait` 函数返回时，会释放对该信号量的锁。
> 
> `sem_wait`、`sem_trywait`、`sem_timedwait` 函数调用成功后返回值均为 0，调用失败返回 ﹣1，可以通过错误码 errno 获得失败原因。
>
> `sem_wait`、`sem_trywait`、`sem_timedwait` 可以被 Linux 信号中断，被信号中断后，函数立即返回，返回值是 ﹣1，错误码 `errno` 为 `EINTR`。

虽然上述函数没有以 `pthread_` 作为前缀，实际使用这个系列的函数时需要链接 `pthread` 库。

我们看一个信号量的具体使用示例：
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
sem_t            mysemaphore;


void* consumer_thread(void* param)
{    
    Task* pTask = NULL;
    while (true)
    {
        if (sem_wait(&mysemaphore) != 0)
            continue;

        if (tasks.empty())
            continue;

        pthread_mutex_lock(&mymutex);   
        pTask = tasks.front();
        tasks.pop_front();
        pthread_mutex_unlock(&mymutex);

        pTask->doTask();
        delete pTask;
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
        sem_post(&mysemaphore);

        taskID ++;

        //休眠1秒
        sleep(1);
    }

    return NULL;
}

int main()
{
    pthread_mutex_init(&mymutex, NULL);
    //初始信号量资源计数为0
    sem_init(&mysemaphore, 0, 0);

    //创建5个消费者线程
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

    sem_destroy(&mysemaphore);
    pthread_mutex_destroy(&mymutex);

    return 0;
}
```
以上代码中我们创建一个生产者线程和 5 个消费者线程，初始信号量计数为 0 代表开始没有可执行任务，因此 5 个消费线程均阻塞在 `sem_wait` 调用处，接着生产者每隔 1 秒产生一个任务，然后通过调用 `sem_post` 将信号量资源计数增加一，此时其中一个线程会被唤醒，然后从任务队列中取出任务，执行任务，由于任务对象是 `new` 出来的，需要 `delete` 掉以避免内存泄露。

有读者可能会奇怪，在调用 `sem_wait` 和 `sem_post` 时会对信号量对象进行加锁和解锁，为什么这里还需要使用一个 `mutex`？这个 `mutex` 是用来保护队列 `tasks` 的，因为多个线程会同时读写之。这个例子类似于银行里多个客户等待柜台有空闲办理取钱业务，每次有空闲的柜台，就可以告诉客户，但是多人同时取钱时，银行的资金总账户增减一定是原子性的。

编译并生成文件 `semaphore` ，然后运行之，输出结果如下：
```
[root@localhost testsemaphore]# g++ -g -o semaphore semaphore.cpp -lpthread
[root@localhost testsemaphore]# ./semaphore 
produce a task, taskID: 0, threadID: 140055260595968
handle a task, taskID: 0, threadID: 140055277381376
produce a task, taskID: 1, threadID: 140055260595968
handle a task, taskID: 1, threadID: 140055277381376
produce a task, taskID: 2, threadID: 140055260595968
handle a task, taskID: 2, threadID: 140055268988672
produce a task, taskID: 3, threadID: 140055260595968
handle a task, taskID: 3, threadID: 140055294166784
produce a task, taskID: 4, threadID: 140055260595968
handle a task, taskID: 4, threadID: 140055302559488
produce a task, taskID: 5, threadID: 140055260595968
handle a task, taskID: 5, threadID: 140055285774080
produce a task, taskID: 6, threadID: 140055260595968
handle a task, taskID: 6, threadID: 140055277381376
produce a task, taskID: 7, threadID: 140055260595968
handle a task, taskID: 7, threadID: 140055268988672
produce a task, taskID: 8, threadID: 140055260595968
handle a task, taskID: 8, threadID: 140055294166784
produce a task, taskID: 9, threadID: 140055260595968
handle a task, taskID: 9, threadID: 140055302559488
...更多输出结果省略...
```
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)。


---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)