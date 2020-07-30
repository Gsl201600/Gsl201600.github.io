---
title: iOS iPhone X适配HomeIndicator相关实践
date: 2019-03-06 14:51:00
tags: 代码库
---

**1. 隐藏HomeIndicator**
一般情况只有视频全屏播放和游戏界面需要设置自动隐藏Home键指示器，隐藏HomeIndicator的方法，如下，
```
- (BOOL)prefersHomeIndicatorAutoHidden {
    return YES;
}
```
在VC 里边重写 prefersHomeIndicatorAutoHidden 返回 YES(默认是NO)，Home指示条就能自动隐藏了，此方法是在控制器push之后就会回调，屏幕若无交互事件响应时，延迟2秒左右会自动隐藏。经过测试发现，只要触摸页面就会重新出现，不操作页面一会儿会自动消失。主要适用于视频类等长时间不对页面做出交互的应用使用。

**2. 屏幕边缘手势冲突**
有时候你的App需要控制从状态栏下拉或者底部栏上滑，这个会跟系统的下拉通知中心手势和上滑控制中心手势冲突。如果你要优先自己处理手势可以将系统手势延迟。方法如下：
```
- (UIRectEdge)preferredScreenEdgesDeferringSystemGestures{
    return UIRectEdgeAll;
}
```
然后其他地方不要修改(比如prefersHomeIndicatorAutoHidden)；就可以像王者荣耀那样,一直显示白条,但是点击一次不会到桌面,也不会到多任务
