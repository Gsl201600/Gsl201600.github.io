---
title: iOS IAP安全性问题汇总
date: 2019-06-26 14:11:00
tags: 代码库
---

**1. 常用的攻击方式**
* 劫持apple server攻击
* 重复验证攻击
* 跨app攻击
* 换价格攻击
* 歧义攻击
* 中间人攻击

**2. 讲解攻击方式及处理**

* 劫持apple server攻击

> 通过dns污染,让客户端通过假的apple_server进行verify,从而认为自己支付成功。这个主要针对客户端验证发货的方式,如果是服务端验证,就没效果了

* 重复验证攻击

> 因为同一个receipt,如果第一次验证成功,那么之后每次验证都会成功。如果服务端没有判重机制,就会导致一个receipt被当做多次充值处理。

> 为了预防这种情况,我们可以将receipt做一次md5得到receipt_md5, 每次发送充值请求的时候就按照receipt_md5判重,如果重复就停止商品发放；或者根据解析得到的信息进行判重

* 跨app攻击

> 通过在别的app中拿到receipt,然后发送到我们app中。因为这个receipt是合法的而且apple不会验证请求的源,所以这个receipt是可以验证通过的

> 对于这种情况,我们可以判断apple verify的返回值apple_callback_data中对应的bundle_id和我们app的bundle_id是否一样来进行验证

* 换价格攻击

> 在同一个app中,用低价商品的receipt伪造购买高价商品。这时候bundle_id和我们app的bundle_id是一致的

> 针对这种情况, 我们可以从apple verify的返回值apple_callback_data中拿到对应的product_id, 并按照product_id来进行充值。不要信任客户端的product_id

* 歧义攻击

> 在iOS6的时候,status=0表示此次支付成功,而现在变为status=0只表示receipt整体上合法。对iOS7即使是一个过期订单,也会返回status=0,如果还按照iOS6的逻辑处理,就会导致假充值

> 针对iOS7,我们应该不只通过status,还要通过in_app中的内容,来决定如何发放商品

> For iOS 6 style transaction receipts, the status code reflects the status of the specific transaction’s receipt.
For iOS 7 style app receipts, the status code is reflects the status of the app receipt as a whole. For example, if you send a valid app receipt that contains an expired subscription, the response is 0 because the receipt as a whole is valid.

* 中间人攻击

> 伪造apple server,将我们的支付请求转发到真的apple_server,拿到合法的receipt,并弄个假的receipt给客户端。这样就拿到一个合法的凭证。利用这个合法的receipt,伪造别人充值的请求,从而达到帮别人充值的目的

> 针对中间人攻击,最重要的是保证a用户的支付receipt,不能被b用户使用。但是apple为了保护隐私,receipt中没有任何用户的个人信息,这就需要我们自己来保证。目前我们可以用加密的手段来做这个保证

* 最后给出一个apple_callback_data的例子
```
{
    "status": 0,
    "environment": "Production",
    "receipt": {
        "download_id": 75017873837267,
        "adam_id": 1149703708,
        "request_date": "2017-01-13 06:57:20 Etc/GMT",
        "app_item_id": 1149703708,
        "original_purchase_date_pst": "2016-11-17 18:57:09 America/Los_Angeles",
        "version_external_identifier": 820252187,
        "receipt_creation_date": "2017-01-13 05:04:52 Etc/GMT",
        "in_app": [
            {
                "is_trial_period": "false",
                "purchase_date_pst": "2017-01-12 21:04:52 America/Los_Angeles",
                "original_purchase_date_pst": "2017-01-12 21:04:52 America/Los_Angeles",
                "product_id": "com.lucky917.live.gold.1.555",
                "original_transaction_id": "350000191094279",
                "original_purchase_date": "2017-01-13 05:04:52 Etc/GMT",
                "original_purchase_date_ms": "1484283892000",
                "purchase_date": "2017-01-13 05:04:52 Etc/GMT",
                "purchase_date_ms": "1484283892000",
                "transaction_id": "350000191094279",
                "quantity": "1"
            }
        ],
        "original_purchase_date_ms": "1479437829000",
        "original_application_version": "26",
        "original_purchase_date": "2016-11-18 02:57:09 Etc/GMT",
        "request_date_ms": "1484290640800",
        "bundle_id": "com.lucky917.ios.Live",
        "receipt_creation_date_pst": "2017-01-12 21:04:52 America/Los_Angeles",
        "application_version": "32",
        "request_date_pst": "2017-01-12 22:57:20 America/Los_Angeles",
        "receipt_creation_date_ms": "1484283892000",
        "receipt_type": "Production"
    }
}
```
