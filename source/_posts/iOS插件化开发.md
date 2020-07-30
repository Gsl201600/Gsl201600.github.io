---
title: iOS 插件化开发(动态库研究)
date: 2019-11-13 17:16:00
tags: 代码库
---

> framework是一种优秀的资源打包方式，我们平时看到的第三方发布的framework大部分都是静态库，苹果对iOS允许使用动态库，但是要利用动态库热更新，由于苹果的审核和签名技术，暂时还是不行，内部使用还是可行的
* 思路：在用户想使用某个功能的时候让其从服务器上将动态库文件下载到本地，然后手动加载动态库，实现功能的的插件化

1. 创建动态库
```
# 头文件部分
#import <Foundation/Foundation.h>

@interface DynamicLlib : NSObject

- (void)doSomething;

@end

# 实现部分
#import "DynamicLlib.h"

@implementation DynamicLlib

- (void)doSomething{
    NSLog(@"doSomething!");
}

@end
```

2. 使用动态库
* 实际过程中动态库是需要从服务器下载并且保存到app的沙盒中的,这边直接模拟已经下载好了动态库并且保存到沙盒中

2.1. 使用NSBundle加载动态库
```
- (IBAction)loadFrameWorkByBundle:(id)sender {
    //从服务器去下载并且存入Documents下(只要知道存哪里即可),事先要知道framework名字,然后去加载
    NSString *frameworkPath = [NSString stringWithFormat:@"%@/Documents/DynamicLlib.framework",NSHomeDirectory()];
    
    NSError *err = nil;
    NSBundle *bundle = [NSBundle bundleWithPath:frameworkPath];
    NSString *str = @"加载动态库失败!";
    if ([bundle loadAndReturnError:&err]) {
        NSLog(@"bundle load framework success.");
        str = @"加载动态库成功!";
    } else {
        NSLog(@"bundle load framework err:%@",err);
    }
}
```

2.2. 使用dlopen加载动态库
```
// 动态库中真正的可执行代码为DynamicLlib.framework/DynamicLlib文件，因此使用dlopen时指定加载动态库的路径为DynamicLlib.framework/DynamicLlib
NSString *documentsPath = [NSString stringWithFormat:@"%@/Documents/DynamicLlib.framework/DynamicLlib",NSHomeDirectory()];
[self dlopenLoadDylibWithPath:documentsPath];
    if (dlopen([path cStringUsingEncoding:NSUTF8StringEncoding], RTLD_NOW) == NULL) { 
        char *error = dlerror(); 
        NSLog(@"dlopen error: %s", error); 
    } else { 
        NSLog(@"dlopen load framework success."); 
 } 
```

2.3. 调用动态库中的方法
```
//调用framework的方法,利用runtime运行时
- (IBAction)callMethodOfFrameWork:(id)sender {
    Class DynamicLlibClass = NSClassFromString(@"DynamicLlib");
    if(DynamicLlibClass){
        //事先要知道有什么方法在这个framework中
        id object = [[DynamicLlibClass alloc] init];
        //由于没有引入相关头文件故通过performSelector调用
        [object performSelector:@selector(doSomething)];
    }else {
        NSLog(@"调用方法失败!");
    }
}
```
