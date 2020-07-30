---
title: iOS 快速定位到系统设置界面
date: 2019-02-01 15:14:00
tags: 代码库
---

```
//定位服务设置界面
NSURL * url = [NSURL URLWithString:@"prefs:root=LOCATION_SERVICES"];
if ([[UIApplication sharedApplication] canOpenURL:url]) {
    [[UIApplication sharedApplication] openURL:url];
}

//定位服务设置界面
@"prefs:root=LOCATION_SERVICES"
//GameCenter设置界面
@"prefs:root=GAME_CENTER"
//WIFI设置界面
@"prefs:root=WIFI"
//通知设置界面
@"prefs:root=NOTIFICATIONS_ID"
//蓝牙设置界面
@"prefs:root=Bluetooth"
//iCloud设置界面
@"prefs:root=CASTLE"

//调用系统邮件发邮件
- (void)gotoEmail:(NSString *)emailAccount{
    NSString * recipients = [NSString stringWithFormat:@"mailto:%@", emailAccount];
    NSString * email = [recipients stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:email]];
}
```
