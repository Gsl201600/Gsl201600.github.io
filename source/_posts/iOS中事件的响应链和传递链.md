---
title: iOS 中事件的响应链和传递链
date: 2019-12-25 16:48:00
tags: 代码库
---

iOS事件链有两条：事件的响应链；`Hit-Testing`事件的传递链
* 响应链：由离用户最近的`view`向系统传递。`initial view` –> `super view` –> ..... –> `view controller` –> `window` –> `Application` –> `AppDelegate`
* 传递链：由系统向离用户最近的view传递。`UIKit` –> `active app's event queue` –> `window` –> `root view` –> ...... –> `lowest view` 

在iOS中只有继承`UIResponder`的对象才能够接收并处理事件，`UIResponder`是所有响应对象的基类，在`UIResponder`类中定义了处理上述各种事件的接口。我们熟悉的`UIApplication、UIViewController、UIWindow`和所有继承自`UIView`的`UIKit`类都直接或间接的继承自`UIResponder`，所以它们的实例都是可以构成响应者链的响应者对象，首先我们通过一张图来简单了解一下事件的传递以及响应
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.12.25.01.png)

1. 传递链
* 事件传递的两个核心方法
```
- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event;   // recursively calls -pointInside:withEvent:. point is in the receiver's coordinate system
- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event;   // default returns YES if point is in bounds
```
* 第一个方法返回的是一个UIView，是用来寻找最终哪一个视图来响应这个事件
* 第二个方法是用来判断某一个点击的位置是否在视图范围内，如果在就返回YES
* 其中UIView不接受事件处理的情况有
```
1. alpha <0.01
2. userInteractionEnabled = NO
3. hidden ＝ YES
```
* 事件传递的流程图
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.12.25.02.png)

* 流程描述
> 1. 我们点击屏幕产生触摸事件，系统将这个事件加入到一个由`UIApplication`管理的事件队列中，`UIApplication`会从消息队列里取事件分发下去，首先传给`UIWindow`
> 2. 在`UIWindow`中就会调用`hitTest:withEvent:`方法去返回一个最终响应的视图
> 3. 在`hitTest:withEvent:`方法中就会去调用`pointInside: withEvent:`去判断当前点击的`point`是否在`UIWindow`范围内，如果是的话，就会去遍历它的子视图来查找最终响应的子视图
> 4. 遍历的方式是使用倒序的方式来遍历子视图，也就是说最后添加的子视图会最先遍历，在每一个视图中都回去调用它的`hitTest:withEvent:`方法，可以理解为是一个递归调用
> 5. 最终会返回一个响应视图，如果返回视图有值，那么这个视图就作为最终响应视图，结束整个事件传递；如果没有值，那么就会将`UIWindow`作为响应者

2. 响应链
* 响应者链流程图
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.12.25.03.png)

* 响应者链的事件传递过程总结如下
> 1. 如果`view`的控制器存在，就传递给控制器处理；如果控制器不存在，则传递给它的父视图
> 2. 在视图层次结构的最顶层，如果也不能处理收到的事件，则将事件传递给`UIWindow`对象进行处理
> 3. 如果`UIWindow`对象也不处理，则将事件传递给`UIApplication`对象
> 4. 如果`UIApplication`也不能处理该事件，则将该事件丢弃

3. 实例场景
* 在一个方形按钮中点击中间的圆形区域有效，而点击四角无效
* 核心思想是在`pointInside: withEvent:`方法中修改对应的区域
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.12.25.04.png)

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    // 如果控件不允许与用用户交互,那么返回nil
    if (!self.userInteractionEnabled || [self isHidden] || self.alpha <= 0.01) {
        return nil;
    }

    //判断当前视图是否在点击范围内
    if ([self pointInside:point withEvent:event]) {
        //遍历当前对象的子视图(倒序)
        __block UIView *hit = nil;
        [self.subviews enumerateObjectsWithOptions:NSEnumerationReverse usingBlock:^(__kindof UIView * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            //坐标转换，把当前坐标系上的点转换成子控件坐标系上的点
            CGPoint convertPoint = [self convertPoint:point toView:obj];
            //调用子视图的hitTest方法，判断自己的子控件是不是最适合的View
            hit = [obj hitTest:convertPoint withEvent:event];
            //如果找到了就停止遍历
            if (hit) *stop = YES;
        }];

        //返回当前的视图对象
        return hit?hit:self;
    }else {
        return nil;
    }
}

// 该方法判断触摸点是否在控件身上，是则返回YES，否则返回NO，point参数必须是方法调用者的坐标系
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {   
    CGFloat x1 = point.x;
    CGFloat y1 = point.y;
    
    CGFloat x2 = self.frame.size.width / 2;
    CGFloat y2 = self.frame.size.height / 2;
    
    //判断是否在圆形区域内
    double dis = sqrt((x1 - x2) * (x1 - x2) + (y1 - y2) * (y1 - y2));
    if (dis <= self.frame.size.width / 2) {
        return YES;
    }
    else{
        return NO;
    }
}
```
