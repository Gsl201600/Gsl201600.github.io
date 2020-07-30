---
title: Core Foundation对象的内存管理
date: 2019-07-18 16:24:00
tags: 代码库
---

* `Foundation`对象和`Core Foundation`对象重要的区别是`ARC`下的内存管理问题，在`非ARC`下两者都需要开发者手动管理内存，没有区别，但是在`ARC`下系统只会自动管理`Foundation`对象的释放，而不支持`Core Foundation`对象的管理，因此在`ARC`下两者进行转换后必须要确定对象是由开发者手动管理还是`ARC`系统管理，否则可能导致内存泄露
* 由于`ARC`不能管理`Core Foundation`对象的生命周期，所以`Core Foundation`对象和`Foundation`对象转换时，需要使用到`__bridge、__bridge_retained和__bridge_transfer`三个转换关键字

1. `__bridge`
`CF`和`OC`对象转化时只涉及对象类型，不涉及对象所有权的转化，他的含义是，不改变对象的管理权所有者，本来由`ARC`管理的`Foundation`对象，转换成`Core Foundation`对象后依旧由`ARC`管理，本来有开发者手动管理的`Core Foundation`对象转换成`Foundation`对象后，继续由开发者手动管理

2. `__bridge_transfer`也可以使用`CFBridgingRelease`
用在将`Core Foundation`对象转换为`Foundation`对象，同时将对象内存管理权交给`ARC`，由`ARC`来代替我们管理内存

3. `__bridge_retained`也可以使用`CFBridgingRetain`
用在将`Foundation`对象转换为`Core Foundation`对象，同时将对象内存管理权交给我们，后续需要使用`CFRelease`或者相关方法来释放对象，需要我们手动来管理内存

* 如下例子为，网络请求中包含特殊字符的处理，报错信息为
`Error Domain=NSURLErrorDomain Code=-1002 "unsupported URL" UserInfo={NSLocalizedDescription=unsupported URL, NSUnderlyingError=0x7fa9b1d06120 {Error Domain=kCFErrorDomainCFNetwork Code=-1002 "(null)"}}`
```
NSString * resultUrl = @"网络请求字段或者地址";
// 处理请求中包含的特殊字符，如“+”
NSString *endResutl = (__bridge_transfer NSString *)CFURLCreateStringByAddingPercentEscapes(kCFAllocatorDefault, (CFStringRef)resultUrl, NULL, CFSTR("+"), kCFStringEncodingUTF8);
```
