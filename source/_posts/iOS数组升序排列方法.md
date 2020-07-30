---
title: iOS 数组升序排列方法
date: 2019-02-22 16:52:00
tags: 代码库
---

```
NSArray *orignalArr = @[@"3", @"12", @"4", @"1", @"4", @"8"];
NSArray *resultArr = [orignalArr sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
    if ([obj1 integerValue] > [obj2 integerValue]) {
        return NSOrderedDescending;
    }else if ([obj1 integerValue] < [obj2 integerValue]){
        return NSOrderedAscending;
    }else{
        return NSOrderedSame;
    }
}];
NSLog(@"升序排列的数组:%@", resultArr);
```
