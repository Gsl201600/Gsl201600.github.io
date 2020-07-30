---
title: iOS KVO
date: 2019-09-18 16:04:00
tags: 代码库
---

`KVO`和`NSNotificationCenter`都是`iOS`中`观察者模式`的一种实现。区别在于，相对于被观察者和观察者之间的关系，`KVO是一对一`的，而`NSNotificationCenter是一对多`的。`KVO`对被监听对象无侵入性，不需要修改其内部代码即可实现监听。
1. `KVO`底层实现
`KVO`是基于`runtime`机制实现的，运用了一个`isa-swizzling`技术。`isa-swizzling`就是`类型混合指针机制`, 将2个对象的`isa`指针互相调换, 就是俗称的黑魔法
* 当某个类的属性对象`第一次被观察`时，系统就会在运行期`动态`地创建`该类的一个派生类`，在这个派生类中重写基类中任何被观察属性的`setter`方法。派生类在被重写的`setter`方法内实现真正的`通知机制`
* 如果原类为`Person`，那么生成的派生类名为`NSKVONotifying_Person`，每个类对象中都有一个`isa指针`指向当前类，当一个类对象第一次被观察，那么系统会偷偷将`isa指针`指向动态生成的派生类，从而在给被监控属性赋值时执行的是派生类的`setter`方法
* 键值观察通知依赖于`NSObject`的两个方法：`willChangeValueForKey:`和`didChangevlueForKey:`；在一个被观察属性发生改变之前，`willChangeValueForKey:`一定会被调用，这就会记录旧的值。而当改变发生后，`didChangeValueForKey:`会被调用，继而`observeValueForKey:ofObject:change:context:`也会被调用
* 补充：`KVO`的这套实现机制中苹果还偷偷重写了`class`方法，让我们误认为还是使用的当前类，从而达到隐藏生成的派生类
2. 如何手动触发`KVO`
手动调用`willChangeValueForKey:`和`didChangeValueForKey:`
示例代码如下：
```
1、自动
//默认返回YES
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key{
    if ([key isEqualToString:@"age"]) {
        return NO;//不观察age属性值得变化
    }
    return YES;
}

2、手动
- (void)setName:(NSString *)name{
    [self willChangeValueForKey:@"name"];
    _name = name;
    [self didChangeValueForKey:@"name"];
}
```

3. 直接修改成员变量会触发`KVO`吗
不会触发`KVO`，添加`KVO`的`Person`实例，其实是`NSKVONotyfing_Person`类在调用`setter`方法，不是调用`Person`的`setter`方法，而是`NSKVONotyfing_Person`的`setter`方法，因为修改成员变量不是`setter`方法赋值
4. 如果在项目中对`Person`类进行了监听，也创建了一个`NSKVONotifying_Person`类，那么会编译通过么
编译通过，因为`KVO`是运行时刻创建的，并不在编译时刻，在编译时刻只有一个`NSKVONotifying_Person`，所以不报错，可以通过，但是此时`KVO`起不了作用
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.09.18.01.png)
