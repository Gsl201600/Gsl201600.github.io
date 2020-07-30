---
title: iOS 如何优化 App 的启动耗时
date: 2020-04-01 16:14:00
tags: 代码库
---

### iOS 的 App 启动时长大概可以这样计算：
* t(`App 总启动时间`) = t1(`main 调用之前的加载时间`) + t2(`main 调用之后的加载时间`)
* t1 = `系统 dylib(动态链接库)`和`自身 App 可执行文件的加载`
* t2 = `main`方法执行之后到`AppDelegate`类中的`application:didFinishLaunchingWithOptions:`方法执行结束前这段时间，主要是构建第一个界面，并完成渲染展示

1. 在`t1`阶段加快`App`启动的建议：
* 尽量使用静态库，减少动态库的使用，动态链接比较耗时，如果要用动态库，尽量将多个`dylib`动态库合并成一个
* 尽量避免对系统库使用`optional linking`，如果`App`用到的系统库在你所有支持的系统版本上都有，就设置为`required`，因为`optional`会有些额外的检查
* 减少`Objective-C Class、Selector、Category`的数量，可以合并或者删减一些`OC`类
* 删减一些无用的静态变量，删减没有被调用到或者已经废弃的方法
* 将不必须在`+load`中做的事情尽量挪到`+initialize`中，`+initialize`是在第一次初始化这个类之前被调用，`+load`在加载类的时候就被调用。尽量将`+load`里的代码延后调用
* 尽量不要用`C++`虚函数，创建虚函数表有开销
* 不要使用` __attribute__((constructor))`将方法显式标记为初始化器，而是让初始化方法调用时才执行。比如使用`dispatch_once()，pthread_once()或 std::once()`
* 在初始化方法中不调用`dlopen()，dlopen()`有性能和死锁的可能性
* 在初始化方法中不创建线程

2. 在`t2`阶段加快`App`启动的建议：
* 尽量不要使用`xib/storyboard`，而是用纯代码作为首页`UI`，如果要用`xib/storyboard`，不要在`xib/storyboard`中存放太多的视图
* 对`application:didFinishLaunchingWithOptions:`里的任务尽量延迟加载或懒加载
* 不要在`NSUserDefaults`中存放太多的数据，`NSUserDefaults`是一个`plist`文件，`plist`文件会被反序列化一次
* 避免在启动时打印过多的`log`，少用`NSLog`，因为每一次`NSLog`的调用都会创建一个新的`NSCalendar`实例
* 为了防止使用`GCD`创建过多的线程，解决方法是创建串行队列，或者使用带有最大并发数限制的`NSOperationQueue`
* 不要在主线程执行`磁盘、网络、Lock或者dispatch_sync、发送消息给其他线程`等操作
