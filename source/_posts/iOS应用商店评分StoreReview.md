---
title: iOS 应用商店评分StoreReview
date: 2019-05-15 14:58:00
tags: 代码库
---

应用中引导用户去进行应用评论常用的方法大概有以下几种：
1. 使用`deepLink`；在`app`地址链接后边拼接上`action=write-review`可以直接跳转到`App Store`应用中对应的应用评价界面进行评价
2. 使用`SKStoreReviewController`；在`iOS 10.3`之后，`iOS`提供了一种新的评价方式，可以不用跳转出应用在应用内就完成应用的星级评价
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.05.15.01.png)

从这段说明里，我们看出官方给出的注意点：
* 该方法在`iOS 10.3`之后才可以使用，所以在使用时需要进行版本判断
* 该方法主要用于申请用户评分，但这个方法不一定会显示`UI`，也就是说即使调用了该方法也不一定会有评级弹窗显示，最终是否有显示主要由`App Store`的相关政策决定，所以这个方法不适用于任何来自按钮或者其他用户直接交互的操作
* 该方法在开发模式下可以弹出交互界面，但是不能进行进行信息提交；在`TestFlight`模式下，调用该方法不会有任何反应
* 同一用户在同一个应用内每年只能提交三次评论，超出次数之后调用该方法就不会有任何反应，但未找到官方文档说明
---------
代码如下：
导入头文件`#import <StoreKit/StoreKit.h>`
```
+ (void)yoStoreReview{
    if (@available(iOS 10.3, *)) {
        if ([SKStoreReviewController respondsToSelector:@selector(requestReview)]){
            //防止键盘遮挡
            [[UIApplication sharedApplication].keyWindow endEditing:YES];
            // iOS10.3+ 直接在App内弹出评分框
            // 此方式苹果允许的调用频率为3次/年
            [SKStoreReviewController requestReview];
        }
    } else {
        // <iOS10.3 跳转AppStore的评论页面
        NSString *appIDStr = [NSString stringWithFormat:@"%@", [YostarUtilits getUserDefaultsForKey:@"APPLEID"]];
        NSString *appStoreReviewStr = [NSString stringWithFormat:@"https://itunes.apple.com/app/id%@?action=write-review", appIDStr];
        [[UIApplication sharedApplication] openURL:[NSURL URLWithString:appStoreReviewStr]];
    }
}
```

[附：[官方文档]  requestReview](https://developer.apple.com/documentation/storekit/skstorereviewcontroller/2851536-requestreview?language=objc)
