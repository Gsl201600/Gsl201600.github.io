---
title: iOS 简单日志系统
date: 2019-07-03 17:00:00
tags: 代码库
---

```
#define YostarDebugLogLevel(level, fmt, ...) \
[YostarDebugLog logLevel:level file:__FILE__ function:__PRETTY_FUNCTION__ line:__LINE__ format:(fmt), ##__VA_ARGS__]

#define YostarDebugLog(fmt, ...) \
YostarDebugLogLevel(YostarDebugLogLevelInfo, (fmt), ##__VA_ARGS__)

#define YostarDebugWarningLog(fmt, ...) \
YostarDebugLogLevel(YostarDebugLogLevelWarning, (fmt), ##__VA_ARGS__)

#define YostarDebugErrorLog(fmt, ...) \
YostarDebugLogLevel(YostarDebugLogLevelError, (fmt), ##__VA_ARGS__)

typedef NS_ENUM(NSUInteger, YostarDebugLogLevel) {
    YostarDebugLogLevelInfo = 1,
    YostarDebugLogLevelWarning,
    YostarDebugLogLevelError
};

@interface YostarDebugLog : NSObject

+ (BOOL)isDebugLogEnabled;

+ (void)enableDebugLog:(BOOL)enableLog;

+ (void)logLevel:(NSInteger)level file:(const char *)file function:(const char *)function line:(NSUInteger)line format:(NSString *)format, ...;
```
```
static BOOL _enableLog;
+ (void)initialize{
    _enableLog = NO;
}

+ (BOOL)isDebugLogEnabled{
    return _enableLog;
}

+ (void)enableDebugLog:(BOOL)enableLog{
    _enableLog = enableLog;
}

static id sharedInstance = nil;
+ (instancetype)sharedInstance{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] init];
    });
    return sharedInstance;
}

+ (void)logLevel:(NSInteger)level file:(const char *)file function:(const char *)function line:(NSUInteger)line format:(NSString *)format, ...{
    @try {
        //参数链表指针
        va_list args;
        //遍历开始
        va_start(args, format);
        NSString *message = [[NSString alloc] initWithFormat:format arguments:args];
        [self.sharedInstance logMessage:message level:level file:file function:function line:line];
        //结束遍历
        va_end(args);
    } @catch (NSException *exception) {
        NSLog(@"⚠️WARN::%@", exception);
    } @finally {

    }
}

- (void)logMessage:(NSString *)message level:(NSInteger)level file:(const char *)file function:(const char *)function line:(NSUInteger)line{
    NSString *logMessage = [NSString stringWithFormat:@"[YostarLog][%@][line:%lu]: %s %s %@", [self descriptionForLevel:level], (unsigned long)line, function, "", message];
    if (_enableLog) {
        NSLog(@"%@", logMessage);
    }
}

- (NSString *)descriptionForLevel:(YostarDebugLogLevel)level{
    NSString *desc = nil;
    switch (level) {
        case YostarDebugLogLevelInfo:
            desc = @"INFO";
            break;
        case YostarDebugLogLevelWarning:
            desc = @"⚠️WARN";
            break;
        case YostarDebugLogLevelError:
            desc = @"❌ERROR";
            break;
        default:
            desc = @"UNKNOW";
            break;
    }
    return desc;
}
```
