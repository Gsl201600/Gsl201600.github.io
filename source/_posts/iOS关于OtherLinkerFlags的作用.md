---
title: iOS 关于Other Linker Flags的作用
date: 2019-12-11 14:43:00
tags: 代码库
---

在用第三方库时，我们常常在Xcode的`Build Settings`下`Other Linker Flags`里面加入`-ObjC`标志，它和`Objective-C`的一个重要特性：类别（`category`)有关
> 根据官方的解释，`Unix`的标准静态库实现和`Objective-C`的动态特性之间有一些冲突：`Objective-C`没有为每个函数（或者方法）定义链接符号，它只为每个类创建链接符号。这样当在一个静态库中使用类别来扩展已有类的时候，链接器不知道如何把类原有的方法和类别中的方法整合起来，就会导致你调用类别中的方法时，出现`"selector not recognized"`，也就是找不到方法定义的错误。

为了解决这个问题，引入了`-ObjC`标志，它的作用就是将静态库中所有的          `Objective-C`代码都加载进来。可以看出，使用`-ObjC`可能会链接很多静态库中未被使用的`Objective-C`代码，极大的增加APP的代码体积。
不要以为这样就可以解决所有问题了，在64位的Mac系统或者iOS系统下，链接器有一个bug，会导致只包含有类别的静态库无法使用`-ObjC`标志来加载文件。
变通方法是使用`-all_load`或者`-force_load`标志，它们的作用都是强制链接器把目标文件都加载进来，即使没有objc代码，不过`-all_load`作用于所有的库，而`-force_load`后面必须要指定具体文件加载的位置

|Flags|位置|作用|
|---|---|---|
|`-ObjC`|`Other Linker Flags`|链接静态库中所有的`Objective-C`代码到APP|
|`-all_load`|`Other Linker Flags`|全加载，链接静态库中所有的代码到APP，无论是`c`、`c++`还是`oc`|
|`-force_load`|`Other Linker Flags`|链接指定静态库中所有的代码到APP，无论是`c`、`c++`还是`oc`|
