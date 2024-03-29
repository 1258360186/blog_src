---
title: 锁使用实践经验总结
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
categories:
  - C/C++ 多线程编程精髓
date: 2021-10-18 14:37:51
---
#### 音乐小港
{% meting "1826924917" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# 锁使用实践经验总结
关于锁的使用，根据我的经验总结如下几点。

## 减少锁的使用
实际开发中能不使用锁尽量不使用锁，当然这不是绝对的，如果使用锁也能满足性能要求，使用也无妨，一般使用了锁的代码会带来如下性能损失：

- 加锁和解锁操作，本身有一定的开销；
- 临界区的代码不能并发执行；
- 进入临界区的次数过于频繁，线程之间对临界区的争夺太过激烈，若线程竞争互斥量失败，就会陷入阻塞，让出 CPU，因此执行上下文切换的次数要远远多于不使用互斥量的版本。
替代锁的方式有很多，如无锁队列。

## 明确锁的范围
看下面这段代码：
```
if(hashtable.is_empty())
{
    pthread_mutex_lock(&mutex);
    htable_insert(hashtable, &elem);
    pthread_mutex_unlock(&mutex);
}
```
读者能看出这段代码的问题吗？代码行 4 虽然对 `hashtable` 的插入使用了锁做保护，但是判断 `hash_table` 是否为空也需要使用锁保护，因此正确的写法应该是：
```
pthread_mutex_lock(&mutex);
if(hashtable.is_empty())
{   
    htable_insert(hashtable, &elem);  
}
pthread_mutex_unlock(&mutex);
```
## 减少锁的粒度
所谓减小锁使用粒度指的是尽量减小锁作用的临界区代码范围，临界区的代码范围越小，多个线程排队进入临界区的时间就会越短。这就类似高速公路上堵车，如果堵车的路段越长，那么后续过来的车辆通行等待时间就会越长。

我们来看两个具体的例子：

示例一
```
void TaskPool::addTask(Task* task)
{
    std::lock_guard<std::mutex> guard(m_mutexList); 
    std::shared_ptr<Task> spTask;
    spTask.reset(task);            
    m_taskList.push_back(spTask);

    m_cv.notify_one();
}
```
上述代码中 `guard` 锁保护 `m_taskList`，仔细分析下这段代码发现，代码行 4、5 和 8 行其实没必要作为临界区内的代码的，因此建议挪到临界区外面去，修改如下：
```
void TaskPool::addTask(Task* task)
{
    std::shared_ptr<Task> spTask;
    spTask.reset(task);

    {
        std::lock_guard<std::mutex> guard(m_mutexList);             
        m_taskList.push_back(spTask);
    }

    m_cv.notify_one();
}
```
修改之后，`guard` 锁的作用范围就是 7 、8 两行了，仅对 `m_taskList.push_back()` 操作做保护，这样锁的粒度就变小了。

示例二
```
void EventLoop::doPendingFunctors()
{
    std::unique_lock<std::mutex> lock(mutex_);
    for (size_t i = 0; i < pendingFunctors_.size(); ++i)
    {
        pendingFunctors_[i]();
    }
}
```
上述代码中 `pendingFunctors_` 是被锁保护的对象，它的类型是 `std::vector<Functor>`，这样的代码效率比较低，必须等当前线程挨个处理完 `pendingFunctors_` 中的元素后其他线程才能操作 `pendingFunctors_` 。修改代码如下：
```
void EventLoop::doPendingFunctors()
{
    std::vector<Functor> functors;

    {
        std::unique_lock<std::mutex> lock(mutex_);
        functors.swap(pendingFunctors_);
    }

    for (size_t i = 0; i < functors.size(); ++i)
    {
        functors[i]();
    }   
}
```
修改之后的代码使用了一个局部变量 `functors`，然后把 `pendingFunctors_` 中的内容倒换到 `functors` 中，这样就可以释放锁了，允许其他线程操作 `pendingFunctors_` ，现在只要继续操作本地对象 `functors` 就可以了，提高了效率。

## 避免死锁的一些建议
- **一个函数中，如果有一个加锁操作，那么一定要记得在函数退出时记得解锁，且每个退出路径上都不要忘记解锁路径。**例如：
```
  void some_func()
  {
      //加锁代码

      if (条件1)
      {
          //其他代码
          //解锁代码
          return;
      } 
      else
      {
          //其他代码
          //解锁代码
          return;
      }


      if (条件2)
      {
          if (条件3)
          {
              //其他代码
              //解锁代码
              return;
          }

          if (条件4)
          {
              //其他代码
              //解锁代码
              return;
          }   
      } 

      while (条件5)
      {
          if (条件6)
          {
              //其他代码
              //解锁代码
              return;
          }
      }
  }
```
上述函数中每个逻辑出口处都需要写上解锁代码。前面也说过，这种逻辑非常容易因为疏忽忘记在某个地方加上解锁代码而造成死锁，因此一般建议使用 `RAII` 技术将加锁和解锁代码封装起来。

- 线程退出时一定要及时释放其持有的锁
实际开发中会因一些特殊需求创建一些临时线程，这些线程执行完相应的任务后就会退出。对于这类线程，如果其持有了锁，一定记得在线程退出时记得释放其持有的锁对象。

- 多线程请求锁的方向要一致，以避免死锁
假设现在有两个锁 A 和 B，线程 1 在请求了锁 A 之后再请求 B，线程 2 在请求了锁 B 后再请求锁 A，这种线程请求锁的方向就不一致了，线程 1 的方向是从 A 到 B，线程 2 的方向是从 B 到 A，多个线程请求锁的方向不一致容易造成死锁。因此建议的方式是线程 1 和 线程 2 请求锁的方向保持一致，要么都从 A 到 B，要么都从 B 到 A。

- 当需要同一个线程重复请求一个锁时，搞清楚你所使用的锁的行为，是递增锁引用计数，还是会阻塞抑或是直接获得锁？
## 避免活锁的一些建议
前面说了避免“死锁”，读者应该能理解，但是这里突然出现了避免“活锁”，我相信很多人看到这个标题一下子就懵了。所谓活锁就是，当多个线程使用 `trylock` 系列的函数时，由于多个线程相互谦让，导致即使在某段时间内锁资源是可用的，也可能导致需要锁的线程拿不到锁。举个生活中的例子，马路上两个人迎面走来，两个人同时往一个方向避让，原来本意是给对方让路，结果还是发生了碰撞。

我们在实际编码时，尽量避免不要过多的线程使用 `trylock` 请求锁，以免出现“活锁”现象，这是对资源的一种浪费。

## 总结
从第 08 节到 第 17 节我们介绍 Windows 和 Linux 操作系统 API 层面上的各种常用多线程同步对象，本节是对它们的使用做了一个规范性和效率性总结。学会使用锁并不难，如何高效地使用它们则是一个不断积累不断总结的过程，希望本节的经验能对读者有帮助。同时，本节介绍锁的注意事项也适用于其他编程语言。

---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)