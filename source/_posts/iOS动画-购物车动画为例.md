---
title: iOS 动画-购物车动画为例
date: 2019-02-15 14:21:00
tags: 代码库
---

### 设定动画
设定动画的属性和说明

|属性|说明|
|-------|:------|
|duration|动画的时长|
|repeatCount|重复的次数。不停重复设置为 HUGE_VALF|
|repeatDuration|设置动画的时间。在该时间内动画一直执行，不计次数。|
|beginTime|指定动画开始的时间。从开始延迟几秒的话，设置为【CACurrentMediaTime() + 秒数】 的方式|
|timingFunction|设置动画的速度变化|
|autoreverses|动画结束时是否执行逆动画|
|fromValue|所改变属性的起始值|
|toValue|所改变属性的结束时的值|
|byValue|所改变属性相同起始值的改变量|
```
transformAnima.fromValue = @(M_PI_2);
transformAnima.toValue = @(M_PI);
transformAnima.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
transformAnima.autoreverses = YES;
transformAnima.repeatCount = HUGE_VALF;
transformAnima.beginTime = CACurrentMediaTime() + 2;
```
### 防止动画结束后回到初始状态
只需设置`removedOnCompletion、fillMode`两个属性就可以了
```
transformAnima.removedOnCompletion = NO;
transformAnima.fillMode = kCAFillModeForwards;
```
### 一些常用的animationWithKeyPath值的总结
|值|说明|使用形式|
|-------|:------|-------|
|transform.scale|比例转化|@(0.8)|
|transform.scale.x|宽的比例|@(0.8)|
|transform.scale.y|高的比例|@(0.8)|
|transform.rotation.x|围绕x轴旋转|@(M_PI)|
|transform.rotation.y|围绕y轴旋转|@(M_PI)|
|transform.rotation.z|围绕z轴旋转|@(M_PI)|
|cornerRadius|圆角的设置|@(50)|
|backgroundColor|背景颜色的变化|(id)[UIColor purpleColor].CGColor|
|bounds|大小，中心不变|[NSValue valueWithCGRect:CGRectMake(0, 0, 200, 200)];|
|position|位置(中心点的改变)|[NSValue valueWithCGPoint:CGPointMake(300, 300)];|
|contents|内容，比如UIImageView的图片|imageAnima.toValue = (id)[UIImage imageNamed:@"to"].CGImage;|
|opacity|透明度|@(0.7)|
|contentsRect.size.width|横向拉伸缩放|@(0.4)最好是0~1之间的|
### 购物车动画
`ShoppingCarAnimationTool.h`定义屏幕信息和block回传
```
#define ScreenWidth [UIScreen mainScreen].bounds.size.width
#define ScreenHeight [UIScreen mainScreen].bounds.size.height

typedef void(^AnimationFinishBlock)(BOOL finishBlock);
// 添加属性
@property (nonatomic, strong) CALayer *layer;
@property (nonatomic, copy) AnimationFinishBlock animationFinishBlock;
// 暴露出来的方法
// 初始化
+ (instancetype)shareTool;

/**
*  开始动画
*  @param view        添加动画的view
*  @param rect        view 的绝对frame
*  @param finishPoint 下落的位置
*  @param completion  动画完成回调
*/
- (void)startAnimationandView:(UIView *)view andRect:(CGRect)rect andFinisnRect:(CGPoint)finishPoint andFinishBlock:(AnimationFinishBlock)completion;

//  摇晃动画
+ (void)shakeAnimation:(UIView *)shakeView;
```
`ShoppingCarAnimationTool.m`实现方法
```
+ (instancetype)shareTool{
    return [[ShoppingCarAnimationTool alloc] init];
}

- (void)startAnimationandView:(UIView *)view andRect:(CGRect)rect andFinisnRect:(CGPoint)finishPoint andFinishBlock:(AnimationFinishBlock)completion{
    //------- 创建CALayer -------//
    _layer = [CALayer layer];
    _layer.contents = view.layer.contents;
    _layer.contentsGravity = kCAGravityResizeAspectFill;
    rect.size.width = 60;
    rect.size.height = 60;  //重置图层尺寸
    _layer.bounds = rect;
    _layer.cornerRadius = rect.size.width/2;
    _layer.masksToBounds = YES; //圆角
    // 添加layer到顶层视图控制器上
    [[UIApplication sharedApplication].delegate.window.layer addSublayer:_layer];

    _layer.position = CGPointMake(rect.origin.x + view.frame.size.width/2, CGRectGetMidY(rect));    //a 点
    //------- 创建移动轨迹 -------/
    UIBezierPath *path = [UIBezierPath bezierPath];
    [path moveToPoint:_layer.position];
    [path addQuadCurveToPoint:finishPoint controlPoint:CGPointMake(ScreenWidth/2, rect.origin.y - 80)];
    //关键帧动画
    CAKeyframeAnimation *pathAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    pathAnimation.path = path.CGPath;
    // 旋转动画
    CABasicAnimation *rotateAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation"];
    rotateAnimation.removedOnCompletion = YES;
    rotateAnimation.fromValue = [NSNumber numberWithFloat:0];
    rotateAnimation.toValue = [NSNumber numberWithFloat:12];
    rotateAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];

    CAAnimationGroup *groups = [CAAnimationGroup animation];
    groups.animations = @[pathAnimation, rotateAnimation];
    groups.duration = 1.2f;
    groups.removedOnCompletion = NO;
    groups.fillMode = kCAFillModeForwards;
    groups.delegate = self;
    [_layer addAnimation:groups forKey:@"group"];
    if (completion) {
        _animationFinishBlock = completion;
    }
}

// 动画结束代理
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag{
    if (anim == [_layer animationForKey:@"group"]) {
        [_layer removeFromSuperlayer];
        _layer = nil;
        if (_animationFinishBlock) {
            _animationFinishBlock(YES);
        }
    }
}

// 上下晃动动画
+ (void)shakeAnimation:(UIView *)shakeView{
    CABasicAnimation *shakeAnimation = [CABasicAnimation animationWithKeyPath:@"transform.translation.y"];
    shakeAnimation.duration = 0.25;
    shakeAnimation.fromValue = [NSNumber numberWithFloat:-5];
    shakeAnimation.toValue = [NSNumber numberWithFloat:5];
    shakeAnimation.autoreverses = YES;
    [shakeView.layer addAnimation:shakeAnimation forKey:nil];
}
```
