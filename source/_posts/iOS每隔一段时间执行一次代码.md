---
title: iOS 每隔一段时间执行一次代码
date: 2019-02-02 13:32:00
tags: 代码库
---

```
//每隔一分钟执行一次打印
// GCD定时器
static dispatch_source_t _timer;
//设置时间间隔
NSTimeInterval period = 60.f;
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
_timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
dispatch_source_set_timer(_timer, dispatch_walltime(NULL, 0), period * NSEC_PER_SEC, 0);
// 事件回调
dispatch_source_set_event_handler(_timer, ^{
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"Count");
        //网络请求 doSomeThing...
    });
});

// 开启定时器
dispatch_resume(_timer);

// 关闭定时器
// dispatch_source_cancel(_timer);
```
