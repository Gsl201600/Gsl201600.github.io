---
title: iOS IAP支付常见问题汇总与解决
date: 2019-06-19 17:35:00
tags: 代码库
---

**1. 获取不到商品信息的原因**
* 沙盒的测试账号和你请求商品信息没有关系
* `iTunes Connect`里面对应账号的`协议、税务和银行业务`信息有没有填完整，填好的应该是这个样子`这个很容易疏忽，务必检查`

![银行税务信息填写完整状态](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.06.19.01.png)
![银行税务信息未填写](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.06.19.02.png)

* 确认证书是否添加`IAP`支付功能`默认创建的证书是包含该项的`
* 确定是真机测试且手机没有越狱`大部分越狱手机也可以测试，深度越狱破坏系统的可能无法调起支付`
* 确定内购商品添加到了需要内购功能的`App`中
* 确定当前运行的`App`的`Bundle ID`和后台配置的`App`的`Bundle ID`是一致的
* 可以尝试先删除旧`App`，再重新编译生成新的，避免新`App`未覆盖错误
* 如果上线后发现线上包请求不到商品信息，一般发生于首次提交`App`或添加新商品，可能是苹果缓存的`bug`，当你的`App`通过审核以后，你发现在生产环境下获取不到商品，这是因为`App`虽然过审核了，但是内购商品还没有正式添加到苹果的服务器里，耐心等待一段时间就可以啦，或者去苹果后台刷新配置商品信息列表，然后等待一天左右时间大概就可以了

**2. 如果请求到了商品信息，也发送了购买请求，但是监听购买结果的方法就是不执行**
* 可以检查一下，是否在工具类初始化的时候，添加了监听，添加监听代码如下
* 注：支付工具类一般用单例模式，避免创建多个对象或者对象提前释放，导致苹果回调不会调用支付失败，或者使用`self`全局化支付工具类对象，不可使支付工具类对象局部变量化
```
#pragma mark - 单例方法
static IAPPayManager* instance = nil;
+ (instancetype)sharedInstance{
  static dispatch_once_t onceToken = 0;
  dispatch_once(&onceToken, ^{
      instance = [[IAPPayManager alloc] init];
  });
  return instance;
}

#pragma mark - 重载初始化方法，注册用于处理支付回调的Observer
- (instancetype)init{
    self = [super init];
    if (self) {
        [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
    }
    return self;
}
```

**3. IAP审核环境**
* 苹果在审核`App`时，只会在`sandbox`环境购买，其产生的购买凭证，也只能连接苹果的测试验证服务器，审核时后台要保证沙盒测试环境开放，以免服务器无法验证通过`IAP`购买，造成`App`审核被拒
* `TestFlight`测试时也是走的`sandbox`环境购买

**4. 只要不是红色的状态都是可以进行支付测试的，元数据丢失是因为，在增加内购项目的时候，没有填写完全，产品ID是唯一的，假如你删除了一个内购项目，那么这个产品ID就不能用了，所以填写要慎重**
![配置内购商品](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.06.19.03.png)

**5. 沙盒测试账号相关**
用沙盒账号测试支付的包，只能是`adhoc`签名证书或者`develop`签名证书打的包，不能是从`AppStore`或者`TestFlight`上下载的，还没上线之前`App`并没有地区之分，沙盒账号随便哪个地区都可以用来测试，弹出的购买提示框会根据当前沙盒账号`AppleID`的地区显示语言的
* 注册沙盒测试账号时，提示报错`Unknown Errors while creating Sandbox Tester, Please check Error Log, email=a***st@qq.com`
解决方案：把你的密码设置的复杂点，比如包含数字、字母混大小写等

**6. 支付时提示`您已购买此App内购买项目。此项目将免费恢复`问题**
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.06.19.04.png)

此提示说明`iTunes`订单被卡住，属于`苹果ID`支付问题，暂时可先选择其他额度进行支付，也可联系苹果的客服人员删除你异常的订单，打开浏览器进入[Apple官方支持](http://www.apple.com/cn/support/contact)

**7. 验证服务器地址和需要的参数说明**

|Key|Value|是否必须|
|---|---|---|
|`receipt-data`|The base64 encoded receipt data|是|
|`password`|Only used for receipts that contain auto-renewable subscriptions. Your app’s shared secret (a hexadecimal string)|否，仅用于自动续订，获取方法见共享密钥附录|
|`exclude-old-transactions`|Only used for iOS7 style app receipts that contain auto-renewable or non-renewing subscriptions. If value is true, response includes only the latest renewal transaction for any subscriptions|否，仅用于自动续订或非续订订阅的iOS 7样式的应用收据|

* 在测试服务器中，发送`receipt`到苹果的测试服务器[https://sandbox.itunes.apple.com/verifyReceipt](https://sandbox.itunes.apple.com/verifyReceipt)验证
* 在正式服务器中`已上线Appstore`，发送`receipt`到苹果的正式服务器[https://buy.itunes.apple.com/verifyReceipt](https://buy.itunes.apple.com/verifyReceipt)验证
* 当我们把应用提交给苹果审核时，苹果也是在`sandbox`环境购买，其产生的购买凭证，也只能连接苹果的测试验证服务器，所以我们可以先发到苹果的正式服务器验证，如果苹果返回`21007`，则再一次连接测试服务器进行验证

**8. 苹果返回状态码**

|Status|描述|
|---|---|
|0|App Store 验证成功|
|21000|App Store不能读取你提供的JSON对象|
|21002|receipt-data属性中的数据格式错误或丢失|
|21003|receipt无法通过验证|
|21004|提供的共享密码与帐户的文件共享密码不匹配|
|21005|receipt服务器当前不可用|
|21006|该收据有效，但订阅已过期，当此状态代码返回到您的服务器时，收据数据也会被解码并作为响应的一部分返回，仅针对自动续订的iOS 6样式交易收据返回|
|21007|receipt是Sandbox receipt，但却发送至生产系统的验证服务|
|21008|receipt是生产receipt，但却发送至Sandbox环境的验证服务|
|21010|此收据无法授权，就像从未进行过购买一样对待|

* 关于苹果服务器验证返回`21004`的问题说明
在购买类型是自动续订时，服务端做验证就要传入这个共享密钥，传入字段为`password`，共享密钥获取见附录，如果你们的商品不是自动续订，建议不要传入该字段，否则传入内容不正确可能会导致苹果返回`21004`
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.06.19.05.png)

**9. 国内连接苹果服务器的稳定性**
开发之初，苹果方就很负责的告知:我们的服务器不稳定。真正开发之后，发现苹果方果然是很负责的，不仅是不稳定，而且足够慢。`app store server`验证一个收据需要`3-6s`时间

**10. 经验总结，如下内容已经过验证**

* 程序加入支付队列使用`SKMutablePayment`和`SKPayment`的区别
两者拥有的属性一样，唯一区别是属性读写权限不同，`SKMutablePayment`属性具有读写权限，`SKPayment`属性只读，如果你要使用`applicationUsername`透传字段，那么就一定要使用`SKMutablePayment`加入支付队列

* 透传字段`applicationUsername`可能返回的是`nil`
在支付完成后，每笔订单都不调用`finishTransaction`，如此测试四五笔订单后，重新启动该应用，苹果自动补单会进行，在有些时候该字段就会为空，需要开发者注意

* `updatedTransactions:`在`App`整个生命周期只会走一次，所以只要不把订单`finishTransaction`掉，重启`App`就会重新走苹果的补单流程(自动调用`updatedTransactions:`注意需要`[[SKPaymentQueue defaultQueue] addTransactionObserver:instance];`添加观察者才可以)，逻辑需要自己根据项目实现

* `SKPaymentTransaction *transaction`属性官方说明
* `transaction.transactionDate`
将订单交易添加到服务器队列的日期，仅当状态为`SKPaymentTransactionStatePurchased`或`SKPaymentTransactionStateRestored`时有效
* `transaction.transactionIdentifier`
`transactionIdentifier`是唯一标识交易支付成功的字符串，此值的格式与收据中的事务`transaction_id`相同，但是值可能不相同，仅当状态为`SKPaymentTransactionStatePurchased`或`SKPaymentTransactionStateRestored`时有效
* `transaction.originalTransaction`
原始交易`id`，仅当状态为`SKPaymentTransactionStateRestored`时有效`有值`
* `transaction.payment.applicationUsername`
获取之前设置的`applicationUsername`
* 注意：凭证验证后返回的`original_transaction_id`和`transaction_id`一般情况下是相同的，只会在恢复购买时不一样

* `transactionReceiptData`可以无限验证通过，也就是说一个凭证可以被校验多次，这是刷单方法之一，需要开发者注意，苹果补单流程返回的`transactionReceiptData`即使同一笔订单也会变

* `transactionReceiptData`验证解析后，`in_app`字段出现为空或者多个购买项目，只要不`finishTransaction`掉订单，下次再支付成功后，返回的`transactionReceiptData`凭证，就是包含之前的购买记录，最近购买的商品会在列表的第一个
* 验证凭证，苹果服务器返回的数据
```
{
  "receipt": {
    "receipt_type": "ProductionSandbox",
    "adam_id": 0,
    "app_item_id": 0,
    "bundle_id": "com.Yo***ights",
    "application_version": "1",
    "download_id": 0,
    "version_external_identifier": 0,
    "receipt_creation_date": "2020-06-01 09:37:57 Etc/GMT",
    "receipt_creation_date_ms": "1591004277000",
    "receipt_creation_date_pst": "2020-06-01 02:37:57 America/Los_Angeles",
    "request_date": "2020-06-01 09:38:55 Etc/GMT",
    "request_date_ms": "1591004335844",
    "request_date_pst": "2020-06-01 02:38:55 America/Los_Angeles",
    "original_purchase_date": "2013-08-01 07:00:00 Etc/GMT",
    "original_purchase_date_ms": "1375340400000",
    "original_purchase_date_pst": "2013-08-01 00:00:00 America/Los_Angeles",
    "original_application_version": "1.0",
    "in_app": [
      {
        "quantity": "1",
        "product_id": "com.yo***thlycard",
        "transaction_id": "10***4780",
        "original_transaction_id": "10***4780",
        "purchase_date": "2020-06-01 09:36:56 Etc/GMT",
        "purchase_date_ms": "1591004216000",
        "purchase_date_pst": "2020-06-01 02:36:56 America/Los_Angeles",
        "original_purchase_date": "2020-06-01 09:36:56 Etc/GMT",
        "original_purchase_date_ms": "1591004216000",
        "original_purchase_date_pst": "2020-06-01 02:36:56 America/Los_Angeles",
        "is_trial_period": "false"
      },
      {
        "quantity": "1",
        "product_id": "com.yo***iteprime1",
        "transaction_id": "10***3950",
        "original_transaction_id": "10***3950",
        "purchase_date": "2020-06-01 09:35:30 Etc/GMT",
        "purchase_date_ms": "1591004130000",
        "purchase_date_pst": "2020-06-01 02:35:30 America/Los_Angeles",
        "original_purchase_date": "2020-06-01 09:35:30 Etc/GMT",
        "original_purchase_date_ms": "1591004130000",
        "original_purchase_date_pst": "2020-06-01 02:35:30 America/Los_Angeles",
        "is_trial_period": "false"
      }
    ]
  },
  "status": 0,
  "environment": "Sandbox"
}
```

[附：[官方文档] Validating Receipts With the App Store](https://developer.apple.com/library/archive/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html#//apple_ref/doc/uid/TP40010573-CH104-SW1)
[附：[官方文档] 配置自动续期订阅共享密钥](https://help.apple.com/app-store-connect/#/devf341c0f01)
[附：iOS开发内购流程](https://www.jianshu.com/p/d9d742e82188)
