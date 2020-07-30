---
title: iOS ImageView加载多图动画
date: 2019-01-10 17:27:00
tags: 代码库
---

```
- (void)startAnimationView{
    //创建UIImageView，添加到界面
    UIImage *image = [UIImage imageNamed:[NSString stringWithFormat:@"fzg_anim_splash_1"]];
    CGSize imageSize = CGSizeMake(image.size.width, image.size.height);
    UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(30, 0, self.view.bounds.size.width - 60, ((self.view.bounds.size.width - 60)*imageSize.height)/imageSize.width)];
    imageView.center = self.view.center;
    imageView.contentMode = UIViewContentModeScaleAspectFit;
    [self.view addSubview:imageView];

    //创建一个数组，数组中按顺序添加要播放的图片（图片为静态的图片）
    NSMutableArray *imageArr = [NSMutableArray array];
    for (int i = 1; i < 32; i++) {
        UIImage *image = [UIImage imageNamed:[NSString stringWithFormat:@"fzg_anim_splash_%d", i]];
        [imageArr addObject:image];
    }
    //把存有UIImage的数组赋给动画图片数组
    imageView.animationImages = imageArr;
    //设置执行一次完整动画的时长
    imageView.animationDuration = 3.5;
    //动画重复次数 （0为重复播放）
    imageView.animationRepeatCount = 1;
    //开始播放动画
    [imageView startAnimating];
    //自动移除动画图层，延迟4秒执行
    [self performSelector:@selector(removeAnimationView) withObject:nil afterDelay:4.f];
}

- (void)removeAnimationView{
    NSLog(@"移除动画");
    [self.view removeFromSuperview];
}
```
