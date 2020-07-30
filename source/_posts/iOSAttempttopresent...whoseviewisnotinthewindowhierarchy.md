---
title: iOS 报错Attempt to present...whose view is not in the window hierarchy!
date: 2019-01-23 18:18:00
tags: 代码库
---

Attempt to present <***ViewController: 0x****> on <ViewController: 0x****> whose view is not in the window hierarchy!
```
#简单来说，就是当前的ViewController还没有被加载完，就调用了另外一个ViewController，这显然是不允许的。
事例：
- (void)viewDidLoad {
    [super viewDidLoad];
    TestVC *vc = [[TestVC alloc]init];
    [self presentViewController:vc animated:NO completion:nil];
}

第一种：在-(void)viewDidAppear:(BOOL)animated；里实现新的控制器跳转。

第二种：延时跳转：
- (void)viewDidLoad {
    [super viewDidLoad];
    [NSTimer scheduledTimerWithTimeInterval:2.0f target:self selector:@selector(jump) userInfo:nil repeats:NO];
}
这种方法可以用，但是不推荐，因为这个延时有可能太长了。

第三种：在主线程上执行指定的方法
- (void)viewDidLoad {
    [super viewDidLoad];
    [self performSelectorOnMainThread:@selector(jump) withObject:nil waitUntilDone:NO];   
}或
[[NSOperationQueue mainQueue] addOperationWithBlock:^{

}];
```
