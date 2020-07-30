---
title: iOS KVC
date: 2019-05-10 15:44:00
tags: 代码库
---

**1. KVC简介**
* 键/值编码中的基本调用是`-valueForKey:`和`-setValue:forKey:`方法
```
NSString *name = [car valueForKey:@"name"];
valueForKey:会首先查找以参数名命名(格式为_name或_isName)的getter方法
如果没有这样的getter方法，它将会在对象内寻找名称格式为_name或name的实例变量
另外KVC会自动装箱和开箱标量值，也就是说，当使用-setValue:forKey:，它自动将标量值(int、float和struct)放入NSNumber或NSValue中；
当时用-valueForKey:时，它自动将标量值从这些对象中取出，仅KVC具有这种自动装箱功能，常规方法调用和属性语法不具备该功能
```

**2. KVC键路径**
* 键路径的基本调用是`-valueForKeyPath:`和`-setValue:forKeyPath:`方法
```
[car setValue:[NSNumber numberWithInt:155] forKeyPath:@"engine.horsepower"];
NSLog(@"horsepower is %@", [car valueForKeyPath:@"engine.horsepower"]);
```

**3. KVC快速运算**
* 键路径不仅能引用对象值，还可以引用一些运算符来进行一些运算，例如能获取一组值的平均值或返回这组值中的最小值和最大值
```
[garage valueForKeyPath:@"cars.@count"];
[garage valueForKeyPath:@"cars.@sum.mileage"];
[garage valueForKeyPath:@"cars.@avg.mileage"];
[garage valueForKeyPath:@"cars.@min.mileage"];
[garage valueForKeyPath:@"cars.@max.mileage"];
```

**4. setter和getter方法命名规则**
* `setter`方法根据它所更改的属性名称来命名，并加上前缀`set`，如：`setName: 、setEngine:` 等
* `getter`方法则是以其返回的属性名称命名，如：`name、engine`等，不要将`get`用作`getter`方法的前缀
* 补充知识：`get`这个词在`Cocoa`中有着特殊的含义，如果`get`出现在`Cocoa`的方法名称中，就意味着这个方法会将你传递的参教作为指针来返回数值。例如，`NSData`中有一个`getBytes:`方法，它的参数就是用来存储字节的内存缓冲区的地址。如果你在存取方法的名称中使用了`get`，那么有经验的`Cocoa`编程人员就会习惯性地将指针当做参数传入这个方法，当他们发现这不过是一个简单的存取方法时就会感到困惑

**5. setValue和setObject的区别**
* `setObject:ForKey:` 是`NSMutableDictionary`特有的；`setValue:ForKey:`是`KVC`的主要方法
* **总结两者的区别：**
* `setObject: forkey:`中`object`是不能够为`nil`
* `setValue: forKey:`中`value`能够为`nil`，但是当`value`为`nil`的时候，会自动调用`removeObject: forKey:`方法
* `setValue: forKey:`中`key`的参数只能够是`NSString`类型
* `setObject: forKey:`的`key`可以是任何类型
* 注意：`setObject: forKey:`对象不能存放`nil`要与下面的这种情况区分：
`[imageDictionary setObject:[NSNullnull] forKey:indexNumber];`
`[NSNull null]`表示的是一个空对象，并不是`nil`
* 当`setValue: forKey:`方法调用者是对象的时候, `setValue: forKey:`方法是在`NSObject`对象中创建的，也就是说所有的`OC`对象都有这个方法，所以可以用于任何类

**6. 正确比较字符串**
* 比较字符串是否相等，应该使用`isEqualToString:`，而不能仅仅比较字符串的指针值；`==`运算符只判断两个字符串的指针数值，而不是它们所指的对象
