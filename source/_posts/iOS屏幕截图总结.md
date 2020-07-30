---
title: iOS 屏幕截图总结
date: 2019-01-29 15:44:00
tags: 代码库
---

1. 按屏幕截图，即全屏截图
```
- (void)doScreenShot{
    // 开启图片上下文
    UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, NO, 0);
    // 获取当前上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    // 截图:实际是把layer上面的东西绘制到上下文中
    [self.view.layer renderInContext:ctx];
    //iOS7+ 推荐使用的方法，代替上述方法
    // [self.view drawViewHierarchyInRect:self.view.frame afterScreenUpdates:YES];
    // 获取截图
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    // 关闭图片上下文
    UIGraphicsEndImageContext();
    // 保存相册
    UIImageWriteToSavedPhotosAlbum(image, NULL, NULL, NULL);
}
```
2. 按内容截屏，即截全图
```
- (void)screenShots{
    UICollectionView *shadowView = self.collectionView;
    // 开启图片上下文
    UIGraphicsBeginImageContextWithOptions(shadowView.contentSize, NO, 0.f);
    // 保存现在视图的位置偏移信息
    CGPoint saveContentOffset = shadowView.contentOffset;
    // 保存现在视图的frame信息
    CGRect saveFrame = shadowView.frame;
    // 把要截图的视图偏移量设置为0
    shadowView.contentOffset = CGPointZero;
    // 设置要截图的视图的frame为内容尺寸大小
    shadowView.frame = CGRectMake(0, 0, shadowView.contentSize.width, shadowView.contentSize.height);
    // 获取当前上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    // 截图:实际是把layer上面的东西绘制到上下文中
    [shadowView.layer renderInContext:ctx];
    //iOS7+ 推荐使用的方法，代替上述方法
    // [shadowView drawViewHierarchyInRect:shadowView.frame afterScreenUpdates:YES];
    // 获取截图
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    // 关闭图片上下文
    UIGraphicsEndImageContext();
    // 将视图的偏移量设置回原来的状态
    shadowView.contentOffset = saveContentOffset;
    // 将视图的frame信息设置回原来的状态
    shadowView.frame = saveFrame;
    // 保存相册
    UIImageWriteToSavedPhotosAlbum(image, NULL, NULL, NULL);
}
```
3. 视频截图
```
// 需要导入AVFoundation头文件
#import <AVFoundation/AVFoundation.h>
// videoURL 视频的URL地址；frameTime 要截取的视频图片帧数
+ (UIImage *)imageWithVideo:(NSURL *)videoURL withVideoCaptureFrame:(NSInteger)frameTime{
    // 根据视频的URL创建AVURLAsset
    AVURLAsset *asset = [[AVURLAsset alloc] initWithURL:videoURL options:nil];
    // 根据AVURLAsset创建AVAssetImageGenerator对象
    AVAssetImageGenerator *gen = [[AVAssetImageGenerator alloc] initWithAsset:asset];
    gen.appliesPreferredTrackTransform = YES;
    // 定义获取第几帧处的视频截图
    // CMTime halfSecond = CMTimeMake(1, 2); //0.5秒
    CMTime time = CMTimeMake(frameTime, 10);
    CMTime actualTime;
    NSError *error = nil;
    // 尝试获取time处的视频截图（实际截屏的时间点保存在actualTime变量中）
    CGImageRef image = [gen copyCGImageAtTime:time actualTime:&actualTime error:&error];
    if (error == nil) {
        // 将CGImageRef转换为UIImage
        UIImage *thumb = [[UIImage alloc] initWithCGImage:image];
        CGImageRelease(image);
        return thumb;
    }else{
        NSLog(@"对视频截图时出现错误: %@", error);
        return nil;
    }
}
```
