---
title: Linux 线程同步之互斥体
comments: true
mathjax: true
tags:
  - Linux
  - Windows
  - C++ 开发
categories:
  - C/C++ 多线程编程精髓
date: 2021-10-18 13:38:19
---
#### 音乐小港
{% meting "1408861641" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Linux 线程同步之互斥体
介绍完 Windows 上的常用多线程同步内核对象后，我们来看一下 Linux 下的线程同步对象。本讲我们介绍一下 Linux 操作系统的 `mutex` 对象。

## Linux mutex 的用法介绍
Linux 互斥体的用法和 Windows 的临界区对象用法很相似，一般也是通过限制多个线程同时执行某段代码来达到保护资源的目的。和接下来要介绍的信号量、条件变量一样，Linux 互斥体都实现在 `NPTL` （`Native POSIX Thread Library`）。在 `NPTL` 中我们使用数据结构 `pthread_mutex_t` 来表示一个互斥体对象（定义于 `pthread.h` 头文件中）。互斥体对象我们可以使用两种方式来初始化：

- 使用 PTHREAD_MUTEX_INITIALIZER 直接给互斥体变量赋值  
  示例代码如下：
  ```
  #include <pthread.h>
  pthread_mutex_t mymutex = PTHREAD_MUTEX_INITIALIZER;
  ```
- 使用 `pthread_mutex_init` 函数初始化
  如果互斥量是动态分配的或者需要给互斥量设置属性，则需要使用 `pthread_mutex_init` 函数来初始化互斥体，这个函数的签名如下：
  ```
  int pthread_mutex_init(pthread_mutex_t* restrict mutex, 
                        const pthread_mutexattr_t* restrict attr);
  ```
  参数 `mutex` 即我们需要初始化的 `mutex` 对象的指针，参数 `attr` 是需要设置的互斥体属性，通常情况下，使用默认属性可以将这个参数设置为 `NULL`，后面会详细介绍每一种属性的用法。如果函数执行成功则会返回 0，如果执行失败则会返回一个具体的错误码信息。

  pthread_mutex_init 代码示例如下：
  ```
  #include <pthread.h>

  pthread_mutex_t mymutex;
  pthread_mutex_init(&mutex, NULL);
  ```
当不再需要一个互斥体对象时，可以使用 `pthread_mutex_destroy` 函数来销毁它， `pthread_mutex_destroy` 函数的签名如下：
```
int pthread_mutex_destroy(pthread_mutex_t* mutex);
```
参数 `mutex` 即我们需要销毁的互斥体对象，如果函数执行成功会返回 0，如果执行失败会返回一个错误码表面出错原因。这里我们需要注意两点：

- **使用 `PTHREAD_MUTEX_INITIALIZER` 初始化的互斥量无须销毁**；
- **不要去销毁一个已经加锁或正在被条件变量使用的互斥体对象**，当互斥量处于已加锁的状态或者正在和条件变量配合使用时，调用 `pthread_mutex_destroy` 函数会返回 `EBUSY` 错误。
以下代码段演示了尝试销毁一个被锁定的 `mutex` 对象：
```
  //test_destroy_locked_mutex.cpp
  #include <pthread.h>
  #include <stdio.h>
  #include <errno.h>

  int main()
  {
      pthread_mutex_t mymutex;
      pthread_mutex_init(&mymutex, NULL);
      int ret = pthread_mutex_lock(&mymutex);

      //尝试对被锁定的mutex对象进行销毁
      ret = pthread_mutex_destroy(&mymutex);
      if (ret != 0)
      {
          if (ret == EBUSY)
              printf("EBUSY\n");
          printf("Failed to destroy mutex.\n");
      }

      ret = pthread_mutex_unlock(&mymutex);
      //尝试对已经解锁的mutex对象进行销毁
      ret = pthread_mutex_destroy(&mymutex);
      if (ret == 0)
          printf("Succeeded to destroy mutex.\n");

      return 0;
  }
```
编译上述代码并执行得到我们期望的结果：
```
  [root@myaliyun codes]# g++ -g -o test_destroy_locked_mutex test_destroy_locked_mutex.cpp -lpthread
  [root@myaliyun codes]# ./test_destroy_locked_mutex 
  EBUSY
  Failed to destroy mutex.
  Succeed to destroy mutex.
```
在实际开发中，如果我们遵循正确的使用 `mutex` 的规范，如创建 `mutex` 对象后再对其加锁，加锁后才对其进行解锁操作，解锁后才做销毁操作，那么编码时一般不用考虑 `pthread_mutex_init`/`pthread_mutex_destroy`/`pthread_mutex_lock`/`pthread_mutex_unlock` 等函数的返回值。

对于互斥体的加锁和解锁操作我们一般使用以下三个函数：
```
int pthread_mutex_lock(pthread_mutex_t* mutex);
int pthread_mutex_trylock(pthread_mutex_t* mutex);
int pthread_mutex_unlock(pthread_mutex_t* mutex);
```
参数 `mutex` 设置为我们需要加锁和解锁的互斥体对象，上述函数执行成功则返回 0；如果执行失败则返回一个错误码表示具体的出错原因。具体错误码，随互斥体对象的属性类型的不同而不同。

设置互斥体对象的属性需要创建一个 `pthread_mutexattr_t` 类型的对象，和互斥体对象一样，需要使用 `pthread_mutexattr_init` 函数初始化之，当不需要这个属性对象时，记得使用 `pthread_mutexattr_destroy` 去销毁它，这两个函数的签名如下：
```
int pthread_mutexattr_init(pthread_mutexattr_t* attr);
int pthread_mutexattr_destroy(pthread_mutexattr_t* attr);
```
使用 `pthread_mutexattr_settype`/`pthread_mutexattr_gettype` 设置或获取想要的属性类型：
```
int pthread_mutexattr_settype(pthread_mutexattr_t* attr, int type);
int pthread_mutexattr_gettype(const pthread_mutexattr_t* restrict attr, int* restrict type);
```
## mutex 锁的类型
属性类型一般有如下取值：

**PTHREAD_MUTEX_NORMAL**（普通锁）
这是互斥体对象的默认属性（即上文中介绍的 `pthread_mutex_init` 第二个函数设置为 `NULL`）。当一个线程对一个普通锁加锁以后，其他线程会阻塞在 `pthread_mutex_lock` 调用处， 直到对互斥体加锁的线程释放了锁，我们来用一段实例代码来验证一下：
```
  #include <pthread.h>
  #include <stdio.h>
  #include <errno.h>
  #include <unistd.h>

  pthread_mutex_t mymutex;
  int             resourceNo = 0;

  void* worker_thread(void* param)
  {
      pthread_t threadID = pthread_self();

      printf("thread start, ThreadID: %d\n", threadID);

      while (true)
      {
          pthread_mutex_lock(&mymutex);

          printf("Mutex lock, resourceNo: %d, ThreadID: %d\n", resourceNo, threadID);
          resourceNo++;

          printf("Mutex unlock, resourceNo: %d, ThreadID: %d\n", resourceNo, threadID);

          pthread_mutex_unlock(&mymutex);

          //休眠1秒
          sleep(1);
      }

      return NULL;
  }

  int main()
  {
      pthread_mutexattr_t mutex_attr;
      pthread_mutexattr_init(&mutex_attr);
      pthread_mutexattr_settype(&mutex_attr, PTHREAD_MUTEX_NORMAL);
      pthread_mutex_init(&mymutex, &mutex_attr);

      //创建5个工作线程
      pthread_t threadID[5];

      for (int i = 0; i < 5; ++i)
      {
          pthread_create(&threadID[i], NULL, worker_thread, NULL);
      }

      for (int i = 0; i < 5; ++i)
      {
          pthread_join(threadID[i], NULL);
      }

      pthread_mutex_destroy(&mymutex);
      pthread_mutexattr_destroy(&mutex_attr);

      return 0;
  }
```
上述代码创建了 5 个工作线程，由于使用了互斥体保护资源 `resourceNo`，因而每次在 `pthread_mutex_lock` 与 `pthread_mutex_unlock` 之间的输出都是连续的，一个线程必须完成了这个工作，其他线程才有机会获得执行这段代码的机会，当一个线程拿到锁后，其他线程会阻塞在 `pthread_mutex_lock` 处。

程序执行结果如下：
```
  [root@localhost testmultithread]# ./test
  thread start, ThreadID: 520349440
  Mutex lock, resourceNo: 0, ThreadID: 520349440
  Mutex unlock, resourceNo: 1, ThreadID: 520349440
  thread start, ThreadID: 545527552
  Mutex lock, resourceNo: 1, ThreadID: 545527552
  Mutex unlock, resourceNo: 2, ThreadID: 545527552
  thread start, ThreadID: 511956736
  Mutex lock, resourceNo: 2, ThreadID: 511956736
  Mutex unlock, resourceNo: 3, ThreadID: 511956736
  thread start, ThreadID: 537134848
  Mutex lock, resourceNo: 3, ThreadID: 537134848
  Mutex unlock, resourceNo: 4, ThreadID: 537134848
  thread start, ThreadID: 528742144
  Mutex lock, resourceNo: 4, ThreadID: 528742144
  Mutex unlock, resourceNo: 5, ThreadID: 528742144
  Mutex lock, resourceNo: 5, ThreadID: 545527552
  Mutex unlock, resourceNo: 6, ThreadID: 545527552
  Mutex lock, resourceNo: 6, ThreadID: 537134848
  Mutex unlock, resourceNo: 7, ThreadID: 537134848
  Mutex lock, resourceNo: 7, ThreadID: 528742144
  Mutex unlock, resourceNo: 8, ThreadID: 528742144
  Mutex lock, resourceNo: 8, ThreadID: 520349440
  Mutex unlock, resourceNo: 9, ThreadID: 520349440
  Mutex lock, resourceNo: 9, ThreadID: 511956736
  Mutex unlock, resourceNo: 10, ThreadID: 511956736
  Mutex lock, resourceNo: 10, ThreadID: 545527552
  Mutex unlock, resourceNo: 11, ThreadID: 545527552
  Mutex lock, resourceNo: 11, ThreadID: 537134848
  Mutex unlock, resourceNo: 12, ThreadID: 537134848
  Mutex lock, resourceNo: 12, ThreadID: 520349440
  Mutex unlock, resourceNo: 13, ThreadID: 520349440
  Mutex lock, resourceNo: 13, ThreadID: 528742144
  Mutex unlock, resourceNo: 14, ThreadID: 528742144
  Mutex lock, resourceNo: 14, ThreadID: 511956736
  Mutex unlock, resourceNo: 15, ThreadID: 511956736
  Mutex lock, resourceNo: 15, ThreadID: 528742144
  Mutex unlock, resourceNo: 16, ThreadID: 528742144
  Mutex lock, resourceNo: 16, ThreadID: 545527552
  Mutex unlock, resourceNo: 17, ThreadID: 545527552
  Mutex lock, resourceNo: 17, ThreadID: 520349440
  Mutex unlock, resourceNo: 18, ThreadID: 520349440
  Mutex lock, resourceNo: 18, ThreadID: 537134848
  Mutex unlock, resourceNo: 19, ThreadID: 537134848
  Mutex lock, resourceNo: 19, ThreadID: 511956736
  Mutex unlock, resourceNo: 20, ThreadID: 511956736
  Mutex lock, resourceNo: 20, ThreadID: 545527552
  Mutex unlock, resourceNo: 21, ThreadID: 545527552
  Mutex lock, resourceNo: 21, ThreadID: 528742144
  Mutex unlock, resourceNo: 22, ThreadID: 528742144
  Mutex lock, resourceNo: 22, ThreadID: 520349440
  Mutex unlock, resourceNo: 23, ThreadID: 520349440
  Mutex lock, resourceNo: 23, ThreadID: 537134848
  Mutex unlock, resourceNo: 24, ThreadID: 537134848
  Mutex lock, resourceNo: 24, ThreadID: 511956736
  Mutex unlock, resourceNo: 25, ThreadID: 511956736
  Mutex lock, resourceNo: 25, ThreadID: 528742144
  Mutex unlock, resourceNo: 26, ThreadID: 528742144
  Mutex lock, resourceNo: 26, ThreadID: 545527552
  Mutex unlock, resourceNo: 27, ThreadID: 545527552
  Mutex lock, resourceNo: 27, ThreadID: 520349440
  Mutex unlock, resourceNo: 28, ThreadID: 520349440
  Mutex lock, resourceNo: 28, ThreadID: 511956736
  Mutex unlock, resourceNo: 29, ThreadID: 511956736
  Mutex lock, resourceNo: 29, ThreadID: 537134848
  Mutex unlock, resourceNo: 30, ThreadID: 537134848
```
一个线程如果对一个已经加锁的普通锁再次使用 `pthread_mutex_lock` 加锁，程序会阻塞在第二次调用 `pthread_mutex_lock` 代码处。测试代码如下：
```
  #include <pthread.h>
  #include <stdio.h>
  #include <errno.h>
  #include <unistd.h>

  int main()
  {
      pthread_mutex_t mymutex;
      pthread_mutexattr_t mutex_attr;
      pthread_mutexattr_init(&mutex_attr);
      pthread_mutexattr_settype(&mutex_attr, PTHREAD_MUTEX_NORMAL);
      pthread_mutex_init(&mymutex, &mutex_attr);

      int ret = pthread_mutex_lock(&mymutex);
      printf("ret = %d\n", ret);

      ret = pthread_mutex_lock(&mymutex);
      printf("ret = %d\n", ret);



      pthread_mutex_destroy(&mymutex);
      pthread_mutexattr_destroy(&mutex_attr);

      return 0;
  }
```
编译并使用 `gdb` 将程序运行起来，程序只输出了一行，我们按 `Ctrl + C` （下文中 ^C 字符）将 `gdb` 中断下来，然后使用 `bt` 命令发现程序确实阻塞在第二个 `pthread_mutex_lock` 函数调用处：
```
  [root@localhost testmultithread]# g++ -g -o test test.cpp -lpthread
  [root@localhost testmultithread]# gdb test
  Reading symbols from /root/testmultithread/test...done.
  (gdb) r
  Starting program: /root/testmultithread/test 
  [Thread debugging using libthread_db enabled]
  Using host libthread_db library "/lib64/libthread_db.so.1".
  ret = 0
  ^C
  Program received signal SIGINT, Interrupt.
  0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
  Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64 libgcc-4.8.5-36.el7.x86_64 libstdc++-4.8.5-36.el7.x86_64
  (gdb) bt
  #0  0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
  #1  0x00007ffff7bc8dcb in _L_lock_883 () from /lib64/libpthread.so.0
  #2  0x00007ffff7bc8c98 in pthread_mutex_lock () from /lib64/libpthread.so.0
  #3  0x00000000004007f4 in main () at ConsoleApplication10.cpp:17
  (gdb) 
```
在这种类型的情况， `pthread_mutex_trylock` 函数如果拿不到锁，不会阻塞，函数会立即返回，并返回 `EBUSY` 错误码。

**PTHREAD_MUTEX_ERRORCHECK**（检错锁）
如果一个线程使用 `pthread_mutex_lock` 对已经加锁的互斥体对象再次加锁，`pthread_mutex_lock` 会返回 `EDEADLK`。

我们验证一下线程对自己已经加锁的互斥体对象再次加锁是什么行为？
```
  #include <pthread.h>
  #include <stdio.h>
  #include <errno.h>
  #include <unistd.h>

  int main()
  {
      pthread_mutex_t mymutex;
      pthread_mutexattr_t mutex_attr;
      pthread_mutexattr_init(&mutex_attr);
      pthread_mutexattr_settype(&mutex_attr, PTHREAD_MUTEX_ERRORCHECK);
      pthread_mutex_init(&mymutex, &mutex_attr);

      int ret = pthread_mutex_lock(&mymutex);
      printf("ret = %d\n", ret);

      ret = pthread_mutex_lock(&mymutex);
      printf("ret = %d\n", ret);
      if (ret == EDEADLK)
      {
          printf("EDEADLK\n");
      }

      pthread_mutex_destroy(&mymutex);
      pthread_mutexattr_destroy(&mutex_attr);

      return 0;
  }
```
编译并运行程序，程序输出结果确实如上面所说：
```
  [root@localhost testmultithread]# g++ -g -o test11 test.cpp -lpthread
  [root@localhost testmultithread]# ./test11
  ret = 0
  ret = 35
  EDEADLK
```
再来看一下，一个线程加锁，其他线程再次加锁的效果：
```
  #include <pthread.h>
  #include <stdio.h>
  #include <errno.h>
  #include <unistd.h>

  pthread_mutex_t mymutex;

  void* worker_thread(void* param)
  {
      pthread_t threadID = pthread_self();

      printf("thread start, ThreadID: %d\n", threadID);

      while (true)
      {
          int ret = pthread_mutex_lock(&mymutex);
          if (ret == EDEADLK)
          {
              printf("EDEADLK, ThreadID: %d\n", threadID);
          } 
          else
              printf("ret = %d, ThreadID: %d\n", ret, threadID);

          //休眠1秒
          sleep(1);
      }

      return NULL;
  }

  int main()
  {
      pthread_mutexattr_t mutex_attr;
      pthread_mutexattr_init(&mutex_attr);
      pthread_mutexattr_settype(&mutex_attr, PTHREAD_MUTEX_ERRORCHECK);
      pthread_mutex_init(&mymutex, &mutex_attr);

      int ret = pthread_mutex_lock(&mymutex);
      printf("ret = %d\n", ret);

      //创建5个工作线程
      pthread_t threadID[5];
      for (int i = 0; i < 5; ++i)
      {
          pthread_create(&threadID[i], NULL, worker_thread, NULL);
      }

      for (int i = 0; i < 5; ++i)
      {
          pthread_join(threadID[i], NULL);
      }

      pthread_mutex_destroy(&mymutex);
      pthread_mutexattr_destroy(&mutex_attr);

      return 0;
  }
```
编译程序，然后使用 `gdb` 运行起来，发现程序并没有有任何输出，按 `Ctrl + C` 中断下来，输入 `info thread` 命令发现工作线程均阻塞在 `pthread_mutex_lock` 函数调用处。操作及输出结果如下：
```
  [root@localhost testmultithread]# g++ -g -o test8 ConsoleApplication8.cpp -lpthread
  [root@localhost testmultithread]# ./test8
  ret = 0
  thread start, ThreadID: -1821989120
  thread start, ThreadID: -1830381824
  thread start, ThreadID: -1838774528
  thread start, ThreadID: -1847167232
  thread start, ThreadID: -1813596416
  ^C
  [root@localhost testmultithread]# gdb test8
  GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-114.el7
  Copyright (C) 2013 Free Software Foundation, Inc.
  License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
  This is free software: you are free to change and redistribute it.
  There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
  and "show warranty" for details.
  This GDB was configured as "x86_64-redhat-linux-gnu".
  For bug reporting instructions, please see:
  <http://www.gnu.org/software/gdb/bugs/>...
  Reading symbols from /root/testmultithread/test8...done.
  (gdb) r
  Starting program: /root/testmultithread/test8 
  [Thread debugging using libthread_db enabled]
  Using host libthread_db library "/lib64/libthread_db.so.1".
  ret = 0
  [New Thread 0x7ffff6fd2700 (LWP 3276)]
  thread start, ThreadID: -151181568
  [New Thread 0x7ffff67d1700 (LWP 3277)]
  thread start, ThreadID: -159574272
  [New Thread 0x7ffff5fd0700 (LWP 3278)]
  thread start, ThreadID: -167966976
  [New Thread 0x7ffff57cf700 (LWP 3279)]
  thread start, ThreadID: -176359680
  [New Thread 0x7ffff4fce700 (LWP 3280)]
  thread start, ThreadID: -184752384
  ^C
  Program received signal SIGINT, Interrupt.
  0x00007ffff7bc7f47 in pthread_join () from /lib64/libpthread.so.0
  Missing separate debuginfos, use: debuginfo-install glibc-2.17-260.el7.x86_64 libgcc-4.8.5-36.el7.x86_64 libstdc++-4.8.5-36.el7.x86_64
  (gdb) bt
  #0  0x00007ffff7bc7f47 in pthread_join () from /lib64/libpthread.so.0
  #1  0x00000000004009e9 in main () at ConsoleApplication8.cpp:50
  (gdb) inf threads
    Id   Target Id         Frame 
    6    Thread 0x7ffff4fce700 (LWP 3280) "test8" 0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
    5    Thread 0x7ffff57cf700 (LWP 3279) "test8" 0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
    4    Thread 0x7ffff5fd0700 (LWP 3278) "test8" 0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
    3    Thread 0x7ffff67d1700 (LWP 3277) "test8" 0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
    2    Thread 0x7ffff6fd2700 (LWP 3276) "test8" 0x00007ffff7bcd4ed in __lll_lock_wait () from /lib64/libpthread.so.0
  * 1    Thread 0x7ffff7fee740 (LWP 3272) "test8" 0x00007ffff7bc7f47 in pthread_join () from /lib64/libpthread.so.0
  (gdb)
```
通过上面的实验，如果互斥体的属性是 `PTHREAD_MUTEX_ERRORCHECK`，当前线程重复调用 `pthread_mutex_lock` 会直接返回 `EDEADLOCK`，其他线程如果对这个互斥体再次调用 `pthread_mutex_lock` 会阻塞在该函数的调用处。

**PTHREAD_MUTEX_RECURSIVE**（嵌套锁）
该属性允许同一个线程对其持有的互斥体重复加锁，每次成功调用 `pthread_mutex_lock` 一次，该互斥体对象的锁引用计数就会增加一次，相反，每次成功调用 `pthread_mutex_unlock` 一次，锁引用计数就会减少一次，当锁引用计数值为 0 时允许其他线程获得该锁，否则其他线程调用 `pthread_mutex_lock` 时尝试获取锁时，会阻塞在那里。这种方式很好理解，这里就不贴示例代码了。

## 总结
我们来总结下 Linux 互斥体对象的使用要点：

- 虽然在上文演示了同一个线程对一个互斥体对象反复进行加锁，但在实际开发中，我们需要用到这种场景的情形非常少；
- 与 Windows 的临界区对象一样，一些有很多出口的逻辑中，为了避免因忘记调用 `pthread_mutex_unlock` 出现死锁或者在逻辑出口处有大量解锁的重复代码出现，建议使用 RAII 技术将互斥体对象封装起来，具体方式在上文中已经介绍过了，这里不再赘述。



---
[点击这里下载课程源代码](https://github.com/balloonwj/gitchat_cppmultithreadprogramming)

来源:[范蠡《C/C++ 多线程编程精髓》](https://gitbook.cn/gitchat/column/5d11e726820bf61799b8277f)