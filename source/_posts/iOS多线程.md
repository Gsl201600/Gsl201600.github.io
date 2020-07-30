---
title: iOS 多线程
date: 2019-01-08 16:44:00
tags: 代码库
---

# 1. 线程与进程
* 进程：进程是资源（CPU、内存等）分配的基本单位，它是程序执行时的一个实例。每个进程之间是独立的，每个进程均运行在其专用且受保护的内存空间内，拥有独立运行所需的全部资源
* 线程：线程是程序执行时的最小单位，它是进程的一个执行流，是CPU调度和分派的基本单位，一个进程可以由很多个线程组成，线程间共享进程的所有资源

# 2. 进程和线程的区别
* 进程是CPU分配资源的最小单位，线程是CPU调用（执行任务）的最小单位
* 一个进程中至少要有一个线程
* 同一个进程内的线程共享进程的资源
* 多进程程序更健壮，多线程程序只要有一个线程死掉，整个进程也死掉了，而一个进程死掉并不会对另外一个进程造成影响，因为进程有自己独立的地址空间

# 3. 多线程的原理
同一时间，CPU只能处理1条线程，只有1条线程在工作（执行）
多线程并发（同时）执行，其实是CPU快速地在多条线程之间调度（切换）
如果CPU调度线程的时间足够快，就造成了多线程并发执行的假象
**思考：如果线程非常非常多，会发生什么情况？**
*CPU会在N多线程之间调度，CPU会累死，消耗大量的CPU资源
每条线程被调度执行的频次会降低（线程的执行效率降低）
![多线程示意图](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.01.08.01.png)

# 4. NSThread、GCD和NSOperation的区别
* NSThread
```
使用更加面向对象
OC语言
简单易用，可直接操作线程对象
线程生命周期程序员管理
```
* GCD
```
旨在替代NSThread等线程技术
充分利用设备的多核
C语言
线程生命周期自动管理
```
* NSOperation
```
使用更加面向对象
OC语言
基于GCD（底层是GCD）
比GCD多了一些更简单实用的功能
线程生命周期自动管理
```

# 5. 创建和启动线程
### 创建、启动线程
```
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];
[thread start];
```
* **创建线程后自动启动线程**
`[NSThread detachNewThreadSelector:@selector(run) toTarget:self withObject:nil];`
* **隐式创建并启动线程**
`[self performSelectorInBackground:@selector(run) withObject:nil];`
* **阻塞（暂停）线程**
```
+ (void)sleepUntilDate:(NSDate *)date;
+ (void)sleepForTimeInterval:(NSTimeInterval)ti;
 ```

# 6. 多线程的安全隐患
**资源共享**
1块资源可能会被多个线程共享，也就是多个线程可能会访问同一块资源
比如多个线程访问同一个对象、同一个变量、同一个文件
当多个线程访问同一块资源时，很容易引发**数据错乱**和**数据安全**问题

# 7. 安全隐患解决 – 互斥锁
**互斥锁使用格式**
`@synchronized(锁对象) {  需要锁定的代码  }`
***注意：锁定1份代码只用1把锁，用多把锁是无效的***
```
互斥锁的优缺点
优点：能有效防止因多线程抢夺资源造成的数据安全问题
缺点：需要消耗大量的CPU资源
互斥锁的使用前提：多条线程抢夺同一块资源
相关专业术语：线程同步
线程同步的意思是：多条线程在同一条线上执行（按顺序地执行任务）
互斥锁，就是使用了线程同步技术
```

# 8. 线程间通信常用方法
```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;
```

# 9. 并发和并行的区别
* 并发：指的是多个事情，在同一时间段内同时发生了
* 并行：指的是多个事情，在同一时间点上同时发生了
* 并发的多个任务之间是互相抢占资源的
* 并行的多个任务之间是不互相抢占资源的
* 只有在多CPU的情况中，才会发生并行。否则，看似同时发生的事情，其实都是并发执行的
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.01.08.02.jpg)
