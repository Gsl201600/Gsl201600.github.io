---
title: iOS 模态跳转推出透明背景方法
date: 2019-03-04 14:39:00
tags: 代码库
---

**1. OS >= iOS 8.0**
```
FZLoginViewController *fzLoginViewController = [[FZLoginViewController alloc] init];
fzLoginViewController.modalPresentationStyle = UIModalPresentationOverCurrentContext;
[[UIApplication sharedApplication].keyWindow.rootViewController presentViewController:fzLoginViewController animated:NO completion:nil];
```
**2. 若系统需兼容7.0 需要加处理**
```
if ([[UIDevice currentDevice].systemVersion floatValue] >= 8.0) {
    controller.modalPresentationStyle = UIModalPresentationOverCurrentContext;
    [self presentViewController:controller animated:YES completion:nil];
} else {
    self.view.window.rootViewController.modalPresentationStyle = UIModalPresentationCurrentContext;
    [self presentViewController:controller animated:NO completion:nil];
    self.view.window.rootViewController.modalPresentationStyle = UIModalPresentationFullScreen;
}
```
