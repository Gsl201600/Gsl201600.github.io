---
title: iOS 后台运行方法
date: 2019-07-19 18:23:00
tags: 代码库
---

应用可以调用`UIApplication`的`beginBackgroundTaskWithExpirationHandler`方法，让应用最多有10分钟的时间在后台长久运行。这个时间可以用来做清理本地缓存、发送统计数据等工作。代码如下：
```
// AppDelegate.h文件
@property (nonatomic, assign) UIBackgroundTaskIdentifier backgroundUpdateTask;

// AppDelegate.m文件
- (void)applicationDidEnterBackground:(UIApplication *)application {
    [self beginBackgroundUpdateTask];
    // 在这里加上你需要长久运行的代码
    [self endBackgroundUpdateTask];
}

- (void)beginBackgroundUpdateTask{
    self.backgroundUpdateTask = [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
        [self endBackgroundUpdateTask];
    }];
}

- (void)endBackgroundUpdateTask{
    [[UIApplication sharedApplication] endBackgroundTask:self.backgroundUpdateTask];
    self.backgroundUpdateTask = UIBackgroundTaskInvalid;
}
```
