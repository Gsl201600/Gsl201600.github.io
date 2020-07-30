---
title: iOS block原理详解
date: 2020-05-13 16:18:00
tags: 代码库
---

1. block本质
* `block`底层就是一个`struct __main_block_impl_0`类型的`结构体`，这个结构体中包含一个`isa`指针，本质上是一个`OC`对象
* `block`是封装了`函数调用`以及`函数调用环境`的`OC`对象

2. block底层结构
`block`底层结构就是`__main_block_impl_0`结构体，内部包含了`impl结构体`和`Desc结构体`以及外部需要访问的`变量`，`block`将需要执行的代码放到一个函数里，`impl`内部的`FuncPtr`指向这个函数的地址，通过地址调用这个函数，就可以执行`block`里面的代码了。`Desc`用来描述`block`，内部的`reserved`作保留，`Block_size`描述`block`占用内存
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.05.13.01.png)

3. block的变量捕获
`局部变量block`访问方式是`值传递`，`auto自动变量`可能会销毁，内存可能会消失，不采用指针访问；
`局部静态变量block`访问方式是`指针传递`，`static变量`一直保存在内存中，指针访问即可;
`全局变量、静态全局变量block`不需要对变量捕获，直接取值
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.05.13.02.png)

```
// block的变量捕获代码解析如下
auto int age = 10;
static int height = 10;    
void (^block)(void) = ^{
    NSLog(@"age is %d,height is %d",age,height);
};        
age = 20;
height = 20;        
block();
-------------------------------------------------
output: age is 10,height is 20

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int age; // 值传递
  int *height; // 指针传递
}
```

4. block的类型

|block类型|环境|存储域|copy操作后|
|---|---|---|---|
|`__NSGlobalBlock__`|没有访问`auto变量`|数据区|什么也不做，类型不改变|
|`__NSStackBlock__`|访问了`auto变量`|栈区|从栈复制到堆，类型改变为`__NSMallocBlock__`|
|`__NSMallocBlock__`|`__NSStackBlock__`调用了`copy`|堆区 |引用计数`+1`，类型不改变|

在`ARC`下`Block`访问`auto变量`时系统默认帮我们进行了`copy`操作，`NSGlobalBlock`访问了`auto变量`时会变成`NSStackBlock`，当`NSStackBlock`进行`copy`操作后会变成`NSMallocBlock`

* 在`ARC`环境下，编译器会根据以下几种情况自动将`栈上的block`复制到`堆上`:
1、`block`作为函数返回值时，比如使用`=`
2、将`block`赋值给`__strong`指针时
3、`block`作为`Cocoa API`中方法名含有`usingBlock`的方法参数时
4、`block`作为`GCD API`的方法参数时

5. 对象类型的auto变量
* 当`block`内部访问了对象类型的`auto变量`时:
如果`block在栈空间`，不论是`ARC还是MRC`环境，不管`外部变量`是`强引用还是弱引用`，`block`都会`弱引用`访问对象
如果`block在堆空间`，如果外部`强引用`，`block`内部也是`强引用`；如果外部`弱引用`，`block`内部也是`弱引用`
* 栈block：
a) 如果`block`是在`栈上`，将不会对`auto变量`产生`强引用`
b) `栈上的block`随时会被销毁，也没必要去强引用其他对象
* 堆block：
**1、如果block被拷贝到堆上**
a) 会调用`block`内部的`copy`函数
b) `copy`函数内部会调用`_Block_object_assign`函数
c) `_Block_object_assign`函数会根据`auto变量`的修饰符`__strong`、`__weak`、`__unsafe_unretained`做出相应的操作，形成`强引用`或者`弱引用`
**2、如果block从堆上移除**
a) 会调用`block`内部的`dispose`函数
b) `dispose`函数内部会调用`_Block_object_dispose`函数
c) `_Block_object_dispose`函数会自动释放引用的`auto变量`(`release`，引用计数`-1`，若为`0`，则销毁)

6. __block
* `__block`修饰符作用：
`__block`可以用于解决`block`内部无法修改`auto变量值`的问题
`__block`不能修饰全局变量、静态变量`static`
* `__block`修饰符原理：
编译器会将`__block`变量包装成一个结构体`__Block_byref_age_0`，结构体内部`*__forwarding`是指向自身的指针，内部还存储着外部`auto变量`的值
`__block`的`forwarding`指针如下图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.05.13.03.png)

`栈上`，`__block`结构体中的`__forwarding`指针`指向自己`，一旦复制到`堆上`，`栈上的__block`结构体中的`__forwarding`指针会指向`堆上的__block`结构体，`堆上__block`结构体中的`__forwarding`还是`指向自己`。假设`age`是`栈上`的变量，`age->__forwarding`会拿到`堆上的__block`结构体，`age->__forwarding->age`会把`20`赋值到`堆上`，不论是`栈上还是堆上的__block`结构体，都能保证`20`赋值到`堆的结构体`里

7. 思考题：block修改NSMutableString、NSMutableArray、NSMutableDictionary，需不需要添加__block
题目如下：以下代码是否可以正确执行
```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSMutableArray *array = [NSMutableArray array];
        void (^block)(void) = ^{
            [array addObject: @"5"];
            [array addObject: @"5"];
            NSLog(@"%@",array);
        };
        block();
    }
    return 0;
}
```
分析：可以正确执行，因为在`block`块中仅仅是使用了`array的内存地址`，往`内存地址`中`添加内容`，并没有修改`arry的内存地址`，因此`array`不需要使用`__block修饰`也可以正确编译。当仅仅是使用`局部变量的内存地址`，而不是`修改`的时候，尽量不要添加`__block`，通过上述分析我们知道一旦添加了`__block`修饰符，系统会自动创建相应的结构体，占用不必要的内存空间
