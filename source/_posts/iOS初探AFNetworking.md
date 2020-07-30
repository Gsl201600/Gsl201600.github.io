---
title: iOS 初探 AFNetworking
date: 2020-07-08 18:06:00
tags: 代码库
---

本文不对`AFNetworking`作全面的解析，仅对比解析一下`2.x`和`3.x`的差异。

1. `AFNetworking`分为如下`5个功能模块`：
* 网络通信模块(`AFURLSessionManager、AFHTTPSessionManger`)
* 网络状态监听模块(`Reachability`)
* 网络通信安全策略模块(`Security`)
* 网络通信信息序列化/反序列化模块(`Serialization`)
* 对于`iOS UIKit`库的扩展(`UIKit`)

2. `AFNetworking 2.x`需要常驻线程而`3.x`不需要常驻线程
`2.x`常驻线程用来`并发请求和处理数据回调`，避免多个网络请求的线程开销(`不用开辟一个线程，就保活一条线程`)；而`3.x`不需要常驻线程是因为`NSURLSession`可以指定回调`delegateQueue`，`NSURLConnection`不行；
`NSURLConnection`的一大痛点就是：发起请求后，需要一直处于`等待回调的状态`。而`3.x`后`NSURLSession`解决了这个问题；`NSURLSession`发起的请求，不再需要在当前线程进行回调，可以指定回调的`delegateQueue`，这样就不用为了等待代理回调方法而保活线程了

3. `3.x`需要设置最大并发数为`1`(`self.operationQueue.maxConcurrentOperationCount = 1`)，`2.x`为什么不需要
功能不一样：`3.x`的`operationQueue`是用来接收`NSURLSessionDelegate`回调的，鉴于一些多线程数据访问的安全性考虑，设置了`maxConcurrentOperationCount = 1`来达到`并发的请求串行的进行回调`的效果。而`2.x`的`operationQueue`是用来添加`operation`进行`并发请求`的，所以不要设置为`1`
**注意：并发数并不等于所开辟的线程数，具体开辟几条线程由系统决定**

4. `3.x`为什么要串行回调
```
- (AFURLSessionManagerTaskDelegate *)delegateForTask:(NSURLSessionTask *)task {
    NSParameterAssert(task);
    AFURLSessionManagerTaskDelegate *delegate = nil;
    [self.lock lock];
    //给所要访问的资源加锁，防止造成数据混乱
    delegate = self.mutableTaskDelegatesKeyedByTaskIdentifier[@(task.taskIdentifier)];
    [self.lock unlock];
    return delegate;
}
```
从代码可以看出，这边对`self.mutableTaskDelegatesKeyedByTaskIdentifier`的访问进行了加锁，目的是`保证多线程环境下的数据安全`。既然加了锁，就算`maxConcurrentOperationCount`不设为`1`，当某个请求正在回调时，下一个请求还是得等待一直到上个请求获取完所要的资源后解锁，所以这边并发回调也是没有意义的。相反多`task`回调导致的多线程并发，还会导致性能的浪费
