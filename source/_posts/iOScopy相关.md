---
title: iOS copy相关
date: 2019-08-08 15:35:00
tags: 代码库
---

1. **strong和copy的区别**
当我们用`@property`来声明属性变量时，编译器会自动为我们生成一个以下划线加属性名命名的实例变量`(@synthesize copyyStr = _copyyStr)`，并且生成其对应的`getter、setter`方法。当我们用`self.copyyStr = originStr`赋值时，会调用coppyStr的`setter`方法，而`_copyyStr = originStr`赋值时给_copyyStr实例变量直接赋值，并不会调用copyyStr的`setter`方法，而在`setter`方法中有一个非常关键的语句
`_copyyStr = [copyyStr copy];`
> 结论：用self.copyyStr = originStr 赋值时，调用copyyStr的setter方法，setter方法对传入的copyyStr做了次深拷贝生成了一个新的对象赋值给_copyyStr，所以_copyyStr指向的地址和对象值都不再和originStr相同

2. **assign与weak**
* assign用来修饰基本数据类型，weak用来修饰OC对象
* assign也能修饰OC对象，但是assign修饰的对象在该对象释放后，其指针依然存在，不会被置为nil，这就造成了一个很严重的问题：出现了野指针。当访问这个野指针时，指向了原地址，如果原地址被回收，就会造成程序的crash
* 用weak来修饰的话，对象释放的时候会把指针置为nil，从而避免了野指针的出现

3. **基本数据类型为什么可以使用assign**
> 这就要扯到堆和栈的问题了，基本数据类型会被分配到栈空间，而栈空间是由系统自动管理分配和释放的，就不会造成野指针的问题

4. **copy和mutableCopy**
* 容器类概念：NSArray、NSDictionary、NSSet为容器类型的对象

* 非容器类总结

|对象类型|不可变对象|可变对象|
|---|---|---|
|copy|浅拷贝|深拷贝|
|mutableCopy|深拷贝|深拷贝|

* 容器类型总结

|对象类型|不可变对象|可变对象|
|---|---|---|
|copy|浅拷贝|深拷贝|
|mutableCopy|深拷贝|深拷贝|
