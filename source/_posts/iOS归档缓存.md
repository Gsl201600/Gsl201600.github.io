---
title: iOS 归档缓存
date: 2019-10-30 17:55:00
tags: 代码库
---

代码如下：

* 头文件定义
```
// 归档缓存内容
+ (void)archiverObject:(id)object byKey:(NSString *)key withPath:(NSString *)path;
// 解归档缓存内容
+ (id)unarchiverObjectByKey:(NSString *)key withPath:(NSString *)path;
```

* 方法实现
```
+ (void)archiverObject:(id)object byKey:(NSString *)key withPath:(NSString *)path{
    //初始化存储对象信息的data
    NSMutableData *data = [NSMutableData data];
    //创建归档工具对象
    NSKeyedArchiver *archiver = [[NSKeyedArchiver alloc] initForWritingWithMutableData:data];
    //开始归档
    [archiver encodeObject:object forKey:key];
    //结束归档
    [archiver finishEncoding];
    //写入本地地址
    NSString *resultStr = [self destPath:path];
    [data writeToFile:resultStr atomically:YES];
}

+ (NSString *)destPath:(NSString *)path{
    NSString *docPath = NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES).lastObject;
    NSString *destPath = [[docPath stringByAppendingPathComponent:@"Caches"] stringByAppendingPathComponent:path];
    NSLog(@"%@", destPath);
    return destPath;
}

+ (id)unarchiverObjectByKey:(NSString *)key withPath:(NSString *)path{
    NSString *resultStr = [self destPath:path];
    NSData *data = [NSData dataWithContentsOfFile:resultStr];
    //创建反归档对象
    NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc] initForReadingWithData:data];
    //接收反归档得到的对象
    id object = [unarchiver decodeObjectForKey:key];
    return object;
}
```

* 使用实例
```
NSDictionary *dict = @{@"1":@"ding", @"2":@"guan", @"3":@"xiong"};
[TestObj archiverObject:dict byKey:@"cache" withPath:@"cache.plist"];

NSDictionary *result = [TestObj unarchiverObjectByKey:@"cache" withPath:@"cache.plist"];
NSLog(@"***::%@", result);
```
