---
title: iOS TouchID and FaceID
date: 2019-02-20 17:31:00
tags: 代码库
---

```
/**
*  .h 继承自 LAContext
*  TouchID/FaceID 状态
*/
typedef NS_ENUM(NSUInteger, XyAuthIDState){

    /**
    *  当前设备不支持TouchID/FaceID
    */
    XyAuthIDStateNotSupport = 0,
    /**
    *  TouchID/FaceID 验证成功
    */
    XyAuthIDStateSuccess = 1,

    /**
    *  TouchID/FaceID 验证失败
    */
    XyAuthIDStateFail = 2,
    /**
    *  TouchID/FaceID 被用户手动取消
    */
    XyAuthIDStateUserCancel = 3,
    /**
    *  用户不使用TouchID/FaceID,选择手动输入密码
    */
    XyAuthIDStateInputPassword = 4,
    /**
    *  TouchID/FaceID 被系统取消 (如遇到来电,锁屏,按了Home键等)
    */
    XyAuthIDStateSystemCancel = 5,
    /**
    *  TouchID/FaceID 无法启动,因为用户没有设置密码
    */
    XyAuthIDStatePasswordNotSet = 6,
    /**
    *  TouchID/FaceID 无法启动,因为用户没有设置TouchID/FaceID
    */
    XyAuthIDStateTouchIDNotSet = 7,
    /**
    *  TouchID/FaceID 无效
    */
    XyAuthIDStateTouchIDNotAvailable = 8,
    /**
    *  TouchID/FaceID 被锁定(连续多次验证TouchID/FaceID失败,系统需要用户手动输入密码)
    */
    XyAuthIDStateTouchIDLockout = 9,
    /**
    *  当前软件被挂起并取消了授权 (如App进入了后台等)
    */
    XyAuthIDStateAppCancel = 10,
    /**
    *  当前软件被挂起并取消了授权 (LAContext对象无效)
    */
    XyAuthIDStateInvalidContext = 11,
    /**
    *  系统版本不支持TouchID/FaceID (必须高于iOS 8.0才能使用)
    */
    XyAuthIDStateVersionNotSupport = 12
};

@interface XyAuthID : LAContext

typedef void (^XyAuthIDStateBlock)(XyAuthIDState state, NSError *error);

/**
* 启动TouchID/FaceID进行验证
* @param describe TouchID/FaceID显示的描述
* @param block 回调状态的block
*/
- (void)xy_showAuthIDWithDescribe:(NSString *)describe block:(XyAuthIDStateBlock)block;

@end
```
```
#import <UIKit/UIKit.h>

#define iPhoneX (UIScreen.mainScreen.bounds.size.width == 375.f && UIScreen.mainScreen.bounds.size.height == 812.f)

@implementation XyAuthID

+ (instancetype)sharedInstance {
    static XyAuthID *instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[XyAuthID alloc] init];
    });
    return instance;
}

- (void)xy_showAuthIDWithDescribe:(NSString *)describe block:(XyAuthIDStateBlock)block {
    if(!describe) {
        if(iPhoneX){
            describe = @"验证已有面容";
        }else{
            describe = @"通过Home键验证已有指纹";
        }
    }

    if (NSFoundationVersionNumber < NSFoundationVersionNumber_iOS_8_0) {

        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"系统版本不支持TouchID/FaceID (必须高于iOS 8.0才能使用)");
            block(XyAuthIDStateVersionNotSupport, nil);
        });

        return;
    }

    LAContext *context = [[LAContext alloc] init];

    // 认证失败提示信息，为 @"" 则不提示
    context.localizedFallbackTitle = @"输入密码";

    NSError *error = nil;

    // LAPolicyDeviceOwnerAuthenticationWithBiometrics: 用TouchID/FaceID验证
    // LAPolicyDeviceOwnerAuthentication: 用TouchID/FaceID或密码验证, 默认是错误两次或锁定后, 弹出输入密码界面（本案例使用）
    if ([context canEvaluatePolicy:LAPolicyDeviceOwnerAuthentication error:&error]) {
        [context evaluatePolicy:LAPolicyDeviceOwnerAuthentication localizedReason:describe reply:^(BOOL success, NSError * _Nullable error) {

            if (success) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    NSLog(@"TouchID/FaceID 验证成功");
                    block(XyAuthIDStateSuccess, error);
                });
            }else if(error){

                if (@available(iOS 11.0, *)) {
                    switch (error.code) {
                        case LAErrorAuthenticationFailed:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID/FaceID 验证失败");
                                block(XyAuthIDStateFail, error);
                            });
                        }
                            break;
                        case LAErrorUserCancel:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID/FaceID 被用户手动取消");
                                block(XyAuthIDStateUserCancel, error);
                            });
                        }
                            break;
                        case LAErrorUserFallback:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"用户不使用TouchID/FaceID,选择手动输入密码");
                                block(XyAuthIDStateInputPassword, error);
                            });
                        }
                            break;
                        case LAErrorSystemCancel:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID/FaceID 被系统取消 (如遇到来电,锁屏,按了Home键等)");
                                block(XyAuthIDStateSystemCancel, error);
                            });
                        }
                            break;
                        case LAErrorPasscodeNotSet:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID/FaceID 无法启动,因为用户没有设置密码");
                                block(XyAuthIDStatePasswordNotSet, error);
                            });
                        }
                            break;
                        //case LAErrorTouchIDNotEnrolled:{
                        case LAErrorBiometryNotEnrolled:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID/FaceID 无法启动,因为用户没有设置TouchID/FaceID");
                                block(XyAuthIDStateTouchIDNotSet, error);
                            });
                        }
                            break;
                        //case LAErrorTouchIDNotAvailable:{
                        case LAErrorBiometryNotAvailable:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID/FaceID 无效");
                                block(XyAuthIDStateTouchIDNotAvailable, error);
                            });
                        }
                            break;
                        //case LAErrorTouchIDLockout:{
                        case LAErrorBiometryLockout:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID/FaceID 被锁定(连续多次验证TouchID/FaceID失败,系统需要用户手动输入密码)");
                                block(XyAuthIDStateTouchIDLockout, error);
                            });
                        }
                            break;
                        case LAErrorAppCancel:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"当前软件被挂起并取消了授权 (如App进入了后台等)");
                                block(XyAuthIDStateAppCancel, error);
                            });
                        }
                            break;
                        case LAErrorInvalidContext:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"当前软件被挂起并取消了授权 (LAContext对象无效)");
                                block(XyAuthIDStateInvalidContext, error);
                            });
                        }
                            break;
                        default:
                            break;
                    }
                } else {
                    // iOS 11.0以下的版本只有 TouchID 认证
                    switch (error.code) {
                        case LAErrorAuthenticationFailed:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID 验证失败");
                                block(XyAuthIDStateFail, error);
                            });
                        }
                            break;
                        case LAErrorUserCancel:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID 被用户手动取消");
                                block(XyAuthIDStateUserCancel, error);
                            });
                        }
                            break;
                        case LAErrorUserFallback:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"用户不使用TouchID,选择手动输入密码");
                                block(XyAuthIDStateInputPassword, error);
                            });
                        }
                            break;
                        case LAErrorSystemCancel:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID 被系统取消 (如遇到来电,锁屏,按了Home键等)");
                                block(XyAuthIDStateSystemCancel, error);
                            });
                        }
                            break;
                        case LAErrorPasscodeNotSet:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID 无法启动,因为用户没有设置密码");
                                block(XyAuthIDStatePasswordNotSet, error);
                            });
                        }
                            break;
                        case LAErrorTouchIDNotEnrolled:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID 无法启动,因为用户没有设置TouchID");
                                block(XyAuthIDStateTouchIDNotSet, error);
                            });
                        }
                            break;
                        case LAErrorTouchIDNotAvailable:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID 无效");
                                block(XyAuthIDStateTouchIDNotAvailable, error);
                            });
                        }
                            break;
                        case LAErrorTouchIDLockout:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"TouchID 被锁定(连续多次验证TouchID失败,系统需要用户手动输入密码)");
                                block(XyAuthIDStateTouchIDLockout, error);
                            });
                        }
                            break;
                        case LAErrorAppCancel:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"当前软件被挂起并取消了授权 (如App进入了后台等)");
                                block(XyAuthIDStateAppCancel, error);
                            });
                        }
                            break;
                        case LAErrorInvalidContext:{
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"当前软件被挂起并取消了授权 (LAContext对象无效)");
                                block(XyAuthIDStateInvalidContext, error);
                            });
                        }
                            break;
                        default:
                            break;
                    }
                }
            }
        }];
    }else{
        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"当前设备不支持TouchID/FaceID");
            block(XyAuthIDStateNotSupport, error);
        });
    }
}

@end
```
