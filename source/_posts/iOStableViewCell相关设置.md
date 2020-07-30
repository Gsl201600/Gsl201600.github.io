---
title: iOS tableViewCell相关设置
date: 2019-02-28 16:26:00
tags: 代码库
---

**1. 去掉底部多余的表格线**
```
[tableView setTableFooterView:[[UIView alloc] initWithFrame:CGRectZero]];
```
**2. 在自定义tableViewCell中设置分割线 顶头显示 self代表cell**
```
if ([self respondsToSelector:@selector(setSeparatorInset:)]) {
    [self setSeparatorInset:UIEdgeInsetsZero];
}

if ([self respondsToSelector:@selector(setLayoutMargins:)]) {
    [self setLayoutMargins:UIEdgeInsetsZero];
}

if([self respondsToSelector:@selector(setPreservesSuperviewLayoutMargins:)]){
    [self setPreservesSuperviewLayoutMargins:NO];
}
```
**3. 代码布局tableview改变cell间隙布局，重写UITableViewCell的frame方法**
```
- (void)setFrame:(CGRect)frame{
    frame.origin.x += 5;
    frame.origin.y += 5;
    frame.size.height -= 5;
    frame.size.width -= 10;
    [super setFrame:frame];
}
```
**4. 动态计算tableviewCell高度**
```
//    获取最底部view 的bottom值
//    NSArray *views = [self.contentView.subviews sortedArrayUsingComparator:^NSComparisonResult(UIView *view1, UIView *view2) {
    //        NSString *top1 = [NSString stringWithFormat:@"%f", view1.frame.origin.y];
    //        NSString *top2 = [NSString stringWithFormat:@"%f", view2.frame.origin.y];
    //        NSComparisonResult result = [top1 compare:top2 options:NSNumericSearch];
    //        NSLog(@"^^^：：%ld", (long)result);
    //
    //        return result == NSOrderedDescending;
    //    }];
NSArray *views = self.contentView.subviews;
UIView * bottomView = views.firstObject;
CGFloat height = bottomView.frame.origin.y+bottomView.frame.size.height;
_model.cellHeight = height + 10;
```
