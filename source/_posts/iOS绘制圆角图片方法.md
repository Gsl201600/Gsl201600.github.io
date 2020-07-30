---
title: iOS 防止离屏渲染绘制圆角图片方法
date: 2019-01-31 15:33:00
tags: 代码库
---

1. 什么是离屏渲染
离屏渲染就是在当前屏幕缓冲区以外，新开辟一个缓冲区进行操作
2. 离屏渲染触发的场景
* 圆角（同时设置`layer.masksToBounds = YES、layer.cornerRadius`大于0）
* 图层蒙版
* 阴影，`layer.shadowXXX`，如果设置了`layer.shadowPath`就不会产生离屏渲染
* 遮罩，`layer.mask`
* 光栅化，`layer.shouldRasterize = YES`
3. 离屏渲染消耗性能的原因
需要创建新的缓冲区，离屏渲染的整个过程，需要多次切换上下文环境，先是从当前屏幕（`On-Screen`）切换到离屏（`Off-Screen`）等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上，又需要将上下文环境从离屏切换到当前屏幕
4. 以下两种绘制圆角图片方法，可以防止离屏渲染
* 贝塞尔曲线方法
```
- (void)drawCornerPicture{
    UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(200, 400, 200, 200)];
    imageView.image = [UIImage imageNamed:@"1"];
    // 开启图片上下文
    // UIGraphicsBeginImageContext(imageView.bounds.size);
    // 一般使用下面的方法
    UIGraphicsBeginImageContextWithOptions(imageView.bounds.size, NO, 0);
    // 绘制贝塞尔曲线
    UIBezierPath *bezierPath = [UIBezierPath bezierPathWithRoundedRect:imageView.bounds cornerRadius:100];
    // 按绘制的贝塞尔曲线剪切
    [bezierPath addClip];
    // 画图
    [imageView drawRect:imageView.bounds];
    // 获取上下文中的图片
    imageView.image = UIGraphicsGetImageFromCurrentImageContext();
    // 关闭图片上下文
    UIGraphicsEndImageContext();
    [self.view addSubview:imageView];
}
```
* Graphics绘制方法
```
- (void)circleImage{
    UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(200, 400, 200, 200)];
    imageView.image = [UIImage imageNamed:@"001.jpeg"];
    // NO代表透明
    UIGraphicsBeginImageContextWithOptions(imageView.bounds.size, NO, 0.0);
    // 获得上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    // 添加一个圆
    CGRect rect = CGRectMake(0, 0, imageView.bounds.size.width, imageView.bounds.size.height);
    CGContextAddEllipseInRect(ctx, rect);
    // 裁剪
    CGContextClip(ctx);
    // 将图片画上去
    //    [imageView drawRect:rect];
    [imageView.image drawInRect:rect];
    imageView.image = UIGraphicsGetImageFromCurrentImageContext();
    // 关闭上下文
    UIGraphicsEndImageContext();
    [self.view addSubview:imageView];
}
```
