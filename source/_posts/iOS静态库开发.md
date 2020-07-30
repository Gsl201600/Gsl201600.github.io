---
title: iOS 静态库开发
date: 2019-03-01 16:18:00
tags: 代码库
---

**本文旨在说明静态库制作中的一些常见问题和特殊处理**
**1. 打包静态库需要的相关问题和设置**
* 静态库中用到分类的需要在项目中设置这个参数：`Other Linker Flags`为`-ObjC`或者`-all_load`
* 静态库中用到了`NSClassFromString`或者`runtime`的`objc_getClass`，但是转换出来的`Class` 一直为`nil`。解决方法：在主工程的`Other Linker Flags`需要添加参数`-ObjC`即可
* 如果Xcode找不到框架的头文件，可能是忘记将它们声明为`public`了
* `Base SDK`指的是当前编译所用的SDK 版本，一般默认为当前xocde的最新版
* `Build Active Architecture Only`设置成`No`
* `Deployment Target`它控制着运行应用需要的最低操作系统版本
* `Skip Install`设置为`Yes`
* `Mach-O Type`静态库设置为`Static Library`，动态库设置为`Dynamic Library`，制作`bundle`文件设置为`Bundle`
* 静态库中最好不要用`xib`，要用的话就将`xib`放到`bundle`文件中编译，然后`xib`就会变成`.nib`的文件
* 如果开发的静态库里面有`C`或者`C++`，在使用的时候需要添加`libc++.tbd`或者`libstdc++.tbd`
* **关于C语言中`Implicit declaration of function ‘XXXX’ is invalid in C99`警告：**C语言是过程化的编程语言，程序执行顺序是从上到下。如果在调用某函数的时候，函数在调用之前没有定义也没有声明，而是在调用之后定义，那么编译时`Implicit declaration of function ‘XXXX’ is invalid in C99`警告就产生了。这是有别于面向对象编程语言的地方

**2. framework中Optional和Required的区别**
* Required：强引用，一定会被加载到内存中，即使不使用也会被加载到内存中
* Optional：弱引用，开始并不会加载，在使用的时候才会加载，会节省加载时的时间。有一些库，如`Social.framework`和`AdSupport.framework`，是在iOS 6之后才被引入的，更新了一些新的特性，如果运行在5.0甚至更低的设备上，这些库不支持，会编译通不过，这时候就要使用弱引用了
* 当你遇到了`dyld:Library not found ……`说明你可能使用了不该有的强引用，根据日志将这个库的引用形式修改一下；或者是使用了动态库，就需要在`Embeded Binaries`选项中添加这个动态库

**3. 如何看一个framework中的二进制文件是静态库还是动态库**
* 使用file命令，如：`$ file /Users/yostar/Desktop/ProjectTest/YostarSDK/ThirdPath/TwitterKit.framework/TwitterKit`；见下面的截图，一个是静态库，一个是动态库
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.01.01.png)

**4. 查看静态库是否支持bitcode**
`$ otool -l /Users/yostar/Desktop/UnityLib/libYostarSDK.a | grep __LLVM`
如果上述命令的输出结果有`__LLVM`，那么就说明，所用的`framework`或`.a`支持设置`Enable bitcode`为`YES`，否则不支持

**5. 静态库相关操作**
* 查看一个库文件支持的指令集：
```
$ lipo -info ./XXXX.a
$ lipo -info ./XXXX.framework/XXXX
```
* 合成指令集：
```
$ lipo -create XXXX_iphoneos.a XXXX_iphonesimulator.a -output XXXX_all.a
$ lipo -create XXXX_iphoneos.framework/XXXX_iphoneos XXXX_iphonesimulator.framework/XXXX_iphonesimulator -output XXXX_all
```
* 拆分特定指令集：
```
$ lipo -thin libname.a armv7(CPU架构名称) -output libname-armv7.a
$ lipo -thin XXXX.framework/XXXX arm64 -output XXXX.framework/XXXX-arm64
```
* **注意**framework和.a处理不同，.a可以直接使用，framework需要做替换处理；framework合并或者拆分完成后，再把输出的文件替换上面simulator文件夹或者iphoneos对应目录下的framework文件

**6. 打包framework之嵌套另一静态库产生类文件重复问题**
将打包好的framework和第三方静态库引入项目，运行，产生两个静态库文件类名重复的问题。如下：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.01.02.png)

这就说明在封装framework时将第三方静态库中的文件给引入了，从而造成两个库中有多个相同类名文件。

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.01.03.png)
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.01.04.png)

这样编译生成的framework就不会和引入的静态库有相同的类文件了

**7. 打包 C,C++文件及和OC混编，接口代码**
* 静态库打包`C`代码
xcode新建文件`YostarUtilits.h`和`YostarUtilits.m`，例子如下：
```
#import <Foundation/Foundation.h>

const char * getIDFA();

@interface YostarUtilits : NSObject

@end
```
```
#import "YostarUtilits.h"

const char * getIDFA(){
    NSString *str = @"123";
    const char *strC = [IDFAStr UTF8String];
    
    char *result = (char *)calloc(10, sizeof(char *));
    if (result) {
        strcpy(result, strC);
    }
    return result;
}

@implementation YostarUtilits

@end
```
* 静态类库打包`C++`代码
xcode新建文件`YostarUtilits.h`和`YostarUtilits.mm`，例子如下：
```
#import <Foundation/Foundation.h>

@interface YostarUtilits : NSObject

@end
```
```
#import "YostarUtilits.h"

#if defined(__cplusplus)
extern "C"
{
#endif

const char * getIDFA(){
    NSString *str = @"123";
    const char *strC = [IDFAStr UTF8String];
    
    char *result = (char *)calloc(10, sizeof(char *));
    if (result) {
        strcpy(result, strC);
    }
    return result;
}
   
#if defined(__cplusplus)
}
#endif

@implementation YostarUtilits

@end
```
