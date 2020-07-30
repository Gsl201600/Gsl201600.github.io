---
title: iOS APNS device token特性
date: 2019-05-09 15:19:00
tags: 代码库
---

* **device token的一些特性：**
1. 开发环境获取的`deviceToken`和发布环境获取的`deviceToken`是不一样的
2. 在一台设备中，`deviceToken`是系统级别的，不同`App`获得的`deviceToken`是相同的
3. `deviceToken`会过期
4. 单个`App`的更新`deviceToken`不会发生改变
5. 当进行备份恢复、或恢复出厂设置之类的操作时，`deviceToken`会发生改变，建议`App`在每次启动时都获取`deviceToken`
6. 用户抹除`iPhone`的数据时，为了保护隐私，`deviceToken`会改变
7. 升级系统`deviceToken`有可能变化，猜测是升级大的系统版本后`deviceToken`会变化
8. 在删除手机上的`App`之后，再次下载安装，`deviceToken`在部分系统上会改变

* **注意：** 推送相关证书只用在推送的后台即服务端使用，工程中只需打开推送相关开关即可，不需要推送证书

* **device token在iOS 13的变化**
1. 在`iOS 13`之前的版本中，大部分这样处理
```
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{
    NSString *dt = [deviceToken description];
    dt = [dt stringByReplacingOccurrencesOfString: @"<" withString: @""];
    dt = [dt stringByReplacingOccurrencesOfString: @">" withString: @""];
    dt = [dt stringByReplacingOccurrencesOfString: @" " withString: @""];
    NSLog(@"**发送给服务器的token字符串***:%@\n", dt);
}
```

2. 在`iOS 13`之后的版本中，必须用以下方法处理（该方法在`iOS 13`之前也兼容）
```
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{
    NSMutableString *deviceTokenString = [NSMutableString string];
    const char *bytes = (const char *)deviceToken.bytes;
    NSInteger count = deviceToken.length;
    for (int i = 0; i < count; i++) {
        [deviceTokenString appendFormat:@"%02x", bytes[i]&0x000000FF];
    }
    NSLog(@"**发送给服务器的token字符串***:%@\n", deviceTokenString);
}
```
或者
```
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{
    if (![deviceToken isKindOfClass:[NSData class]]) return;
    const unsigned *tokenBytes = (const unsigned *)[deviceToken bytes];
    NSString *hexToken = [NSString stringWithFormat:@"%08x%08x%08x%08x%08x%08x%08x%08x",
                          ntohl(tokenBytes[0]), ntohl(tokenBytes[1]), ntohl(tokenBytes[2]),
                          ntohl(tokenBytes[3]), ntohl(tokenBytes[4]), ntohl(tokenBytes[5]),
                          ntohl(tokenBytes[6]), ntohl(tokenBytes[7])];
    NSLog(@"**发送给服务器的token字符串***:%@\n",hexToken);
}
```
