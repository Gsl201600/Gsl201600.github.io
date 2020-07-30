---
title: iOS 单例篇
date: 2018-12-26 11:55:00
tags: 代码库
---

## 1. 单例在ARC中的实现
**ARC中单例实现步骤**
* 在类的内部提供一个static修饰的全局变量
* 提供一个类方法，方便外界访问
* 重写`+allocWithZone`方法，保证永远都只为单例对象分配一次内存空间
* 严谨起见，重写`-copyWithZone`方法和`-MutableCopyWithZone`方法

**ARC中单例代码实现**
```
#import "Tools.h"

@implementation Tools
// 创建静态对象 防止外部访问
static Tools *_instance = nil;
+(instancetype)allocWithZone:(struct _NSZone *)zone{
    //    @synchronized (self) {
    //        // 为了防止多线程同时访问对象，造成多次分配内存空间，所以要加上线程锁
    //        if (_instance == nil) {
    //            _instance = [super allocWithZone:zone];
    //        }
    //        return _instance;
    //    }
    // 也可以使用一次性代码
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        if (_instance == nil) {
            _instance = [super allocWithZone:zone];
        }
    });
    return _instance;
}
// 为了使实例易于外界访问 我们一般提供一个类方法
// 类方法命名规范 share类名|default类名|类名
+(instancetype)shareTools{
    //return _instance;
    // 最好用self 用Tools他的子类调用时会出现错误
    return [[self alloc] init];
}
// 为了严谨，也要重写copyWithZone 和 mutableCopyWithZone
-(id)copyWithZone:(NSZone *)zone{
    return _instance;
}
-(id)mutableCopyWithZone:(NSZone *)zone{
    return _instance;
}
```
## 2. 单例在MRC中的实现
**MRC单例实现步骤**
* 在类的内部提供一个static修饰的全局变量
* 提供一个类方法，方便外界访问
* 重写`+allocWithZone`方法，保证永远都只为单例对象分配一次内存空间
* 严谨起见，重写`-copyWithZone`方法和`-MutableCopyWithZone`方法
* 重写release方法
* 重写retain方法
* 建议在retainCount方法中返回一个最大值

**配置MRC环境**
* 注意ARC不是垃圾回收机制，是编译器特性
* 配置MRC环境：build setting ->搜索automatic ref->修改为N0
MRC中单例代码实现
* 配置好MRC环境之后，在ARC代码基础上重写下面的三个方法即可
```
-(oneway void)release{

}
-(instancetype)retain{
    return _instance;
}
-(NSUInteger)retainCount{
    return MAXFLOAT;
}
```
## 3. 单例的另一种实现
直接告诉外面 alloc、new、copy、mutableCopy方法不可以直接调用，否则编译不过。
```
+ (instancetype)alloc __attribute__((unavailable("call sharedInstance instead")));
+ (instancetype)new __attribute__((unavailable("call sharedInstance instead")));
- (instancetype)copy __attribute__((unavailable("call sharedInstance instead")));
- (instancetype)mutableCopy __attribute__((unavailable("call sharedInstance instead")));
```
