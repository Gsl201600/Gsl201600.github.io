---
title: iOS 内购项目的App Store推广
date: 2020-05-27 16:50:00
tags: 代码库
---

`iOS 11`以后的用户可以在`App Store`内的下载页面内直接购买应用的内购商品，这项功能苹果称作做`Promoting In-App Purchases`，如果你的`App`需要在`App Store`推广自己的内购商品，则需要在`App Store`推广里上传推广用的图像，另外苹果也在`iOS11 SDK`里面新增了从`App Store`购买内购项目跳转到`App`的新方法

1. 选择`推广App内购买项目`的好处
* 提高展示促销机会，在产品页面上，开发者可一次性推广多达`20个App`内购买项目
* 提高下载量，`App`内购买项目的推广还能促进`App`的下载量。如果用户尚未安装`App`，在点击购买`App内购项目`时，会引导其先下载

2. 如何推广`App内购买项目`
* 在`App Store Connect`中为准备推广的`App`内购买项目上传宣传图像。该图像不但会显示在`App Store`产品页面，也可能显示在搜索结果中。如果入选精品推荐，它更可能显示在`Today、游戏今日亮点、App今日亮点`中。当`App`内购买项目显示在`App Store`产品页面以外的地方（如搜索结果中），`App`图标会显示在外框的左下方，所以要确保设计的宣传图像不会被外框遮盖，注意重要细节不要放在左下角，不建议在图像上叠加文字

* `App Store`后台内购项目的配置，默认情况下，推广的`App`内购买项目将面向所有设备显示，即使它们没有安装`App`。①在工具栏中，点按`功能`，然后在左列中点按`App 内购买项目`。②点需要修改的`App内购买项目`，然后前往`App Store 推广`部分。③配置`面向所有 App Store 用户显示，即使是没有安装该 App 的用户`复选框设置。④点按`存储`。⑤在左列中点按`App Store 推广`，勾选需要推广的项目
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.05.27.01.png)

3. 开发者需注意
* `iOS11以上`用户可见，所以产品需要针对`iOS11以上`系统兼容
* 开发者显示`App`内购买项目推广后，不一定被显示在苹果搜索结果中，只是可能
* 苹果明确规定只有`除消耗型 App 内购买项目`会显示在搜索结果中
* 产品提供订阅获取收益十分可观，从而苹果针对`自动续期订阅`也十分看重，并且为其提供了相关设置方法及运营手段的介绍，所以开发者们可重点尝试`App`内购买项目中`自动续期订阅`形式展示，能够得到苹果更多认可及展示
* 游戏开发者可考虑利用关卡设置`App 内购买项目`的形式吸引苹果关注，不同关卡有不同的`App 内购买项目（非消耗型）`或完整体验需付费的形式，类似这类`App`苹果是给予鼓励的
* 用户直接在`App`下载页面购买内购商品，这就涉及到从`App Store`跳转到自己`App`，所以苹果在`SKPaymentTransactionObserver`新增了一个代理方法：
`- (BOOL)paymentQueue:(SKPaymentQueue *)queue shouldAddStorePayment:(SKPayment *)payment forProduct:(SKProduct *)product`
这个代理函数是在`App Store`发起购买的时候会有回调，用户如果在`App`下载页面点击购买你推广的内购商品，如果用户已经安装过你的`App`则会直接跳转你的`App`并调用上述代理方法；如果用户还没有安装你的`App`那么就会去下载你的`App`，下载完成之后系统会推送一个通知，如果用户点击该通知就会跳转到你的`App`并且调用上面的代理方法
上面的代理方法返回`YES`则表示跳转到你的`App`，`IAP`继续完成交易，如果返回`NO`则表示推迟或者取消购买，实际开发中因为可能还需要用户登录自己的账号、生成订单等，一般都是返回`NO`，之后自己手动把代理方法里面返回的`SKPayment`加入支付队列，然后在按照自己的支付、验证逻辑完成支付

4. 测试
苹果提了测试方法，就是修改下面的链接地址，然后在`safari`浏览器打开，就可以测试从`App Store`发起购买了。其中链接中的`bundleId`修改为你自己应用的`bundleId`，`productId`修改为你创建的商品的`id`，如：`itms-services://?action=purchaseIntent&bundleId=bundleId&productIdentifier=productId`

[附：iOS 内购总结](https://xiaovv.me/2018/05/03/My-iOS-In-App-Purchase-Summarize)
[附：[官方文档] What's New in StoreKit](https://developer.apple.com/videos/play/wwdc2017/303)
[附：[官方文档] App Store Connect 帮助](https://help.apple.com/app-store-connect)
[附：[官方文档] 推广您的 App 内购买项目](https://developer.apple.com/cn/app-store/promoting-in-app-purchases)
[附：[官方文档] Testing Promoted In-App Purchases](https://developer.apple.com/documentation/storekit/in-app_purchase/testing_promoted_in-app_purchases)
[附：Promoting-In-App-PurchasesDemo](https://github.com/Gsl201600/Promoting-In-App-PurchasesDemo.git)
