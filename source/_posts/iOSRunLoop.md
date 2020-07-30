---
title: iOS RunLoop
date: 2020-01-17 14:57:00
tags: 代码库
---

### RunLoop概念
* `RunLoop`是通过内部维护的`事件循环(Event Loop)`来对`事件/消息`进行管理的一个对象
* 没有消息处理时，休眠以避免资源占用；有消息需要处理时，立刻被唤醒

1. 为什么`main`函数不会退出
```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
`UIApplicationMain`内部默认开启了主线程的`RunLoop`，并执行了一段无限循环的代码（不是简单的`for循环`或`while循环`）`UIApplicationMain`函数一直没有返回，不断地接收处理消息以及等待休眠，所以运行程序之后，会保持持续运行状态

###  RunLoop结构体
* `Source1` : 基于`Port`的线程间通信
* `Source0` : 触摸事件、`PerformSelector`
* `Timer` : 定时器
* `Observer` : 监听器，用于监听`RunLoop`的状态

### RunLoop和线程
* 线程和`RunLoop`是一一对应的，其映射关系是保存在一个全局的`Dictionary`里，线程作为`key`，`RunLoop`作为`value`
* 自己创建的线程默认是没有开启`RunLoop`的
* `runloop`在第一次获取时被创建，在线程结束时被销毁
* 对于主线程来说，`runloop`在程序一启动就默认创建好了
* 对于子线程来说，`runloop`是懒加载的，只有当我们使用的时候才会创建，所以在子线程用定时器要注意：确保子线程的`runloop`被创建，不然定时器不会回调

1. 怎么创建一个常驻线程
* 为当前线程开启一个`RunLoop`（第一次调用`[NSRunLoop currentRunLoop]`方法时，实际是会先去创建一个`RunLoop`）
* 向当前`RunLoop`中添加一个`Port/Source`等维持`RunLoop`的事件循环（如果`RunLoop`的`mode`中一个`item`都没有，`RunLoop`会退出）
* 启动该`RunLoop`
```
@autoreleasepool {
  NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
  [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
  [runLoop run];
}
```

2. 输出下边代码的执行顺序
```
NSLog(@"1");

dispatch_async(dispatch_get_global_queue(0, 0), ^{
  NSLog(@"2");
  [self performSelector:@selector(test) withObject:nil afterDelay:10];
  NSLog(@"3");
});

NSLog(@"4");

- (void)test{
  NSLog(@"5");
}
```
答案是`1423`，`test`方法并不会执行
原因是：如果是带`afterDelay`的延时函数，会在内部创建一个`NSTimer`，然后添加到当前线程的`RunLoop`中，也就是如果当前线程没有开启`RunLoop`，该方法会失效
那么我们改成:
```
dispatch_async(dispatch_get_global_queue(0, 0), ^{
  NSLog(@"2");
  [[NSRunLoop currentRunLoop] run];
  [self performSelector:@selector(test) withObject:nil afterDelay:10];
  NSLog(@"3");
});
```
`test`方法依然不执行
原因是：如果`RunLoop`的`mode`中一个`item`都没有，`RunLoop`会退出
即在调用`RunLoop`的`run`方法后，由于其`mode`中没有添加任何`item`去维持`RunLoop`的事件循环，`RunLoop`随即还是会退出，所以我们自己启动`RunLoop`，一定要在添加`item`后
```
dispatch_async(dispatch_get_global_queue(0, 0), ^{
  NSLog(@"2");
  [self performSelector:@selector(test) withObject:nil afterDelay:10];
  [[NSRunLoop currentRunLoop] run];
  NSLog(@"3");
});
```
