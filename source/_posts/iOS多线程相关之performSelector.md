---
title: iOS 多线程相关之performSelector、死锁
date: 2020-01-08 17:28:00
tags: 代码库
---

1. `performSelector`
```
//在当前线程延迟1s执行，响应了OC语言的动态性：延迟到运行时才绑定方法
[self performSelector:@selector(aaa) withObject:nil afterDelay:1];
// 回到主线程，waitUntilDone:是否将该回调方法执行完再执行后面的代码
// 如果为YES：就必须等回调方法执行完成之后才能执行后面的代码，说白了就是阻塞当前的线程
// 如果是NO：就是不等回调方法结束，不会阻塞当前线程
[self performSelectorOnMainThread:@selector(aaa) withObject:nil waitUntilDone:YES];
// 开辟子线程
[self performSelectorInBackground:@selector(aaa) withObject:nil];
//在指定线程执行
[self performSelector:@selector(aaa) onThread:[NSThread currentThread] withObject:nil waitUntilDone:YES];
```
* **需要注意的是：**如果是带`afterDelay`的延时函数，会在内部创建一个`NSTimer`，然后添加到当前线程的`Runloop`中。也就是如果当前线程没有开启`runloop`，该方法会失效。在子线程中，需要启动`runloop`(注意调用顺序)
```
[self performSelector:@selector(aaa) withObject:nil afterDelay:1];
[[NSRunLoop currentRunLoop] run];
```
* `performSelector:withObject:`只是一个单纯的消息发送，和时间没有一点关系。所以不需要添加到子线程的`Runloop`中也能执行
* 下面代码片段的`test`方法会去执行吗？
```
dispatch_async(dispatch_get_global_queue(0, 0), ^{
  [self performSelector:@selector(test:) withObject:nil afterDelay:0];
});
```
这里的`test`方法是不会去执行的，原因在于`- (void)performSelector: withObject: afterDelay:`这个方法要创建提交任务到`runloop`上的，而`gcd`底层创建的线程是默认没有开启对应`runloop`的，所有这个方法就会失效。
而如果将`dispatch_get_global_queue`改成主队列，由于主队列所在的主线程是默认开启了`runloop`的，就会去执行(将`dispatch_async`改成同步，因为同步是在当前线程执行，那么如果当前线程是主线程，`test`方法也是会去执行的)

2. 死锁
* 死锁就是队列引起的循环等待，一个比较常见的死锁例子:主队列同步
```
- (void)viewDidLoad {
  [super viewDidLoad];
  dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"deallock");
  });
}
```
* 在主线程中运用主队列同步，也就是把任务放到了主线程的队列中。同步对于任务是立刻执行的，那么当把任务放进主队列时，它就会立马执行，只有执行完这个任务，`viewDidLoad`才会继续向下执行。而`viewDidLoad`和任务都是在主队列上的，由于队列的`先进先出`原则，任务又需等待`viewDidLoad`执行完毕后才能继续执行，`viewDidLoad`和这个任务就形成了相互循环等待，就造成了死锁。
想避免这种死锁，可以将同步改成异步`dispatch_async`或者将`dispatch_get_main_queue`换成其他串行或并行队列，都可以解决
* 同样，下边的代码也会造成死锁：
```
dispatch_queue_t serialQueue = dispatch_queue_create("test", DISPATCH_QUEUE_SERIAL);
dispatch_async(serialQueue, ^{
  dispatch_sync(serialQueue, ^{
    NSLog(@"deadlock");
  });
});
```
* 外面的函数无论是同步还是异步都会造成死锁。这是因为里面的任务和外面的任务都在同一个`serialQueue`队列内，又是同步，这就和上边主队列同步的例子一样造成了死锁。
解决方法也和上边一样，将里面的同步改成异步`dispatch_async`或者将`serialQueue`换成其他串行或并行队列，都可以解决
