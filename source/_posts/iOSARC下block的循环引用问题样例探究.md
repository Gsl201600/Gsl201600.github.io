---
title: iOS ARC下block的循环引用问题样例探究
date: 2019-01-02 15:47:28
tags: 代码库
---

下面的六种情况，是否会产生内存泄漏。
```
//情况一
- (void)case1 {
    NSLog(@"case 1 Click");
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        self.name = @"case 1";
    });
}

//情况二
- (void)case2 {
    NSLog(@"case 2 Click");
    __weak typeof(self) weakSelf = self;
    [self.teacher requestData:^(NSData *data) {
        typeof(weakSelf) strongSelf = weakSelf;
        strongSelf.name = @"case 2";
    }];
}

//情况三
- (void)case3 {
    NSLog(@"case 3 Click");
    [self.teacher requestData:^(NSData *data) {
        self.name = @"case 3";
    }];
}

//情况四
- (void)case4 {
    NSLog(@"case 4 Click");
    [self.teacher requestData:^(NSData *data) {
        self.name = @"case 4";
        self.teacher = nil;
    }];
}

//情况五
- (void)case5 {
    NSLog(@"case 5 Click");
    Teacher *t = [[Teacher alloc] init];
    [t requestData:^(NSData *data) {
        self.name = @"case 5";
    }];
}

//情况六
- (void)case6 {
    NSLog(@"case 6 Click");
    [self.teacher callCase6BlackEvent];
    self.teacher.case6Block = ^(NSData *data) {
        self.name = @"case 6";
        //下面两句代码任选其一
        self.teacher = nil;
        //        self.teacher.case6Block = nil;
    };
}
```
> **分析：**
**情况一：**执行了dealloc，不泄露，此情况虽然是block，但未形成保留环block -> self
**情况二：**执行了dealloc，不泄露，此情况就是内存泄漏后的一般处理了 self ->teacher ->block ->strongSelf，后面那个strongSelf和原来的self并没有直接关系，因为strongSelf是通过weakSelf得来的，而weakSelf又没有强引用原来的self
**情况三：**未执行dealloc，内存泄漏，此情况就是最典型的循环引用了，形成保留环无法释放，self ->teacher ->block ->self
**情况四：**执行了dealloc，不泄露，虽然也是保留环，但通过最后一句，使self不再强引用teacher，打破了保留环
**情况五：**执行了dealloc，不泄露，未形成保留环 t ->block ->self
**情况六：**执行了dealloc，不泄露，最后两句代码任选其一即可防止内存泄漏，self.teacher 或者 case6Block 置为空都可以打破 retain cycle
PS: 虽然情况四、情况六的写法都可以防止内存泄漏，不过为了统一，个人建议最好还是按照普通写法即情况二的写法。
