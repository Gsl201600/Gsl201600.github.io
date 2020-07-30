---
title: iOS UIButton之改变有效点击区域
date: 2019-08-07 14:51:00
tags: 代码库
---

* 解决方案
通过重写`- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;`
以改变按钮的有效点击区域
```
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event{
    if (_qi_clickAreaReduceValue > 0) {
        if (_qi_clickAreaReduceValue >= CGRectGetWidth(self.bounds)/2) {
            _qi_clickAreaReduceValue = CGRectGetWidth(self.bounds)/2;
        }
        CGRect bounds = CGRectInset(self.bounds, _qi_clickAreaReduceValue, _qi_clickAreaReduceValue);
        return CGRectContainsPoint(bounds, point);
    }

    // 获取bounds 实际大小
    CGRect bounds = self.bounds;
    // 若热区小于 44 * 44 则放大热区 否则保持原大小不变
    CGFloat widthDelta = MAX(44.f - bounds.size.width, 0.f);
    CGFloat heightDelta = MAX(44.f - bounds.size.height, 0.f);
    // 扩大bounds
    bounds = CGRectInset(bounds, -0.5 * widthDelta, -0.5 * heightDelta);
    // 点击的点在新的bounds 中 就会返回YES
    return CGRectContainsPoint(bounds, point);
}
```
