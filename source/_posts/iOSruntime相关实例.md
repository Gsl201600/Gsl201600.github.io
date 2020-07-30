---
title: iOS runtime 相关实例
date: 2019-04-19 10:39:00
tags: 代码库
---

**避免按钮快速点击多次相应问题**

* 创建UIButton 分类

```
//
//  UIButton+time.h
//

#import <UIKit/UIKit.h>

@interface UIButton (time)

/* 防止button重复点击，设置间隔 */
@property (nonatomic, assign) NSTimeInterval acceptEventInterval;

@end
```
```
//
//  UIButton+time.m
//

#import "UIButton+time.h"
#import <objc/runtime.h>

@implementation UIButton (time)

static const char *UIButton_acceptEventInterval = "UIButton_acceptEventInterval";
static const char *UIButton_acceptEventTime = "UIButton_acceptEventTime";

// get方法 获取时间间隔
- (NSTimeInterval)acceptEventInterval{
    return [objc_getAssociatedObject(self, UIButton_acceptEventInterval) doubleValue];
}

// set方法 赋值时间间隔
- (void)setAcceptEventInterval:(NSTimeInterval)mm_acceptEventInterval{
    objc_setAssociatedObject(self, UIButton_acceptEventInterval, @(mm_acceptEventInterval), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

// get方法 获取时间
- (NSTimeInterval)acceptEventTime{
    return [objc_getAssociatedObject(self, UIButton_acceptEventTime) doubleValue];
}

// set方法 赋值时间
- (void)setAcceptEventTime:(NSTimeInterval)mm_acceptEventTime{
    objc_setAssociatedObject(self, UIButton_acceptEventTime, @(mm_acceptEventTime), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

/**
class_addMethod:如果发现方法已经存在，会失败返回，也可以用来做检查用,我们这里是为了避免源方法没有实现的情况;如果方法没有存在,我们则先尝试添加被替换的方法的实现
1.如果返回成功:则说明被替换方法没有存在.也就是被替换的方法没有被实现,我们需要先把这个方法实现,然后再执行我们想要的效果,用我们自定义的方法去替换被替换的方法. 这里使用到的是'class_replaceMethod'这个方法. class_replaceMethod本身会尝试调用class_addMethod和method_setImplementation，所以直接调用class_replaceMethod就可以了)
2.如果返回失败:则说明被替换方法已经存在.直接将两个方法的实现交换即
*/
//分类中重写load方法，实现方法的交换（只要能让其执行一次方法交换语句，load再合适不过了）
+ (void)load{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        //获取这两个方法
        Method systemMethod = class_getInstanceMethod(self, @selector(sendAction:to:forEvent:));
        SEL sysSEL = @selector(sendAction:to:forEvent:);

        Method myMethod = class_getInstanceMethod(self, @selector(mm_sendAction:to:forEvent:));
        SEL mySEL = @selector(mm_sendAction:to:forEvent:);

        //添加方法进去
        BOOL isAddMethod = class_addMethod(self, sysSEL, method_getImplementation(myMethod), method_getTypeEncoding(myMethod));
        //如果添加方法成功
        if (isAddMethod) {
            class_replaceMethod(self, mySEL, method_getImplementation(systemMethod), method_getTypeEncoding(systemMethod));
        }else{
            method_exchangeImplementations(systemMethod, myMethod);
        }
    });
    /*-----以上主要是实现两个方法的互换,load是gcd的只shareinstance，果断保证执行一次-------*/
}

- (void)mm_sendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event{
    if ([NSDate date].timeIntervalSince1970 - self.acceptEventTime < self.acceptEventInterval) {
        return;
    }

    if (self.acceptEventInterval > 0) {
        self.acceptEventTime = [NSDate date].timeIntervalSince1970;
    }

    [self mm_sendAction:action to:target forEvent:event];
}

@end
```
