---
title: iOS 系统自带分享
date: 2019-05-22 14:49:00
tags: 代码库
---

* **注意：**国行手机无法使用系统自带的facebook分享，国行手机facebook被阉割导致分享失败。
```
/**
*  分享
*  多图分享，items里面直接放图片
*  分享链接
*  NSString *textToShare = @"mq分享";
*  UIImage *imageToShare = [UIImage imageNamed:@"imageName"];
*  NSURL *urlToShare = [NSURL URLWithString:@"https:www.baidu.com"];
*  NSArray *items = @[urlToShare,textToShare,imageToShare];
*/
- (void)yoShare:(NSArray *)items success:(ShareSuccessBlock)successBlock fail:(ShareFailBlock)failBlock{
    if (0 == items.count) {
        return;
    }
    UIActivityViewController *activityVC = [[UIActivityViewController alloc] initWithActivityItems:items applicationActivities:nil];
    if (@available(iOS 11.0, *)) {
    //UIActivityTypeMarkupAsPDF是在iOS 11.0 之后才有的
        activityVC.excludedActivityTypes = @[UIActivityTypeMessage, UIActivityTypeMail, UIActivityTypeOpenInIBooks, UIActivityTypeMarkupAsPDF];
    }else if (@available(iOS 9.0, *)){
    //UIActivityTypeOpenInIBooks是在iOS 9.0 之后才有的
        activityVC.excludedActivityTypes = @[UIActivityTypeMessage, UIActivityTypeMail, UIActivityTypeOpenInIBooks];
    }else{
        activityVC.excludedActivityTypes = @[UIActivityTypeMessage, UIActivityTypeMail];
    }
    activityVC.completionWithItemsHandler = ^(UIActivityType  _Nullable activityType, BOOL completed, NSArray * _Nullable returnedItems, NSError * _Nullable activityError) {
        if (completed) {
            if (successBlock) {
                NSNumber *codeNum = [NSNumber numberWithInteger:YoErrorCodeSuccess];
                NSDictionary *result = @{RCODEKEY:codeNum, RMSGKEY:SuccessMsg, METHODKEY:SysShareMethod};
                successBlock(result);
            }
        }else{
            if (failBlock) {
                NSNumber *codeNum = [NSNumber numberWithInteger:YoErrorCodeShareFail];
                NSDictionary *result = @{RCODEKEY:codeNum, RMSGKEY:ShareFailMsg, METHODKEY:SysShareMethod};
                failBlock(result);
            }
        }
    };
    //这儿一定要做iPhone与iPad的判断，因为这儿只有iPhone可以present，iPad需pop，所以这儿actVC.popoverPresentationController.sourceView = self.view;在iPad下必须有，不然iPad会crash，self.view你可以换成任何view，你可以理解为弹出的窗需要找个依托。
    UIViewController *vc = [UIApplication sharedApplication].keyWindow.rootViewController;
    if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) {
        activityVC.popoverPresentationController.sourceView = vc.view;
        activityVC.popoverPresentationController.sourceRect = CGRectMake([UIScreen mainScreen].bounds.size.width/2, [UIScreen mainScreen].bounds.size.height, 0, 0);
        [vc presentViewController:activityVC animated:YES completion:nil];
    }else{
        [vc presentViewController:activityVC animated:YES completion:nil];
    }
}
```
