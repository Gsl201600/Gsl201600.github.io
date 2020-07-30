---
title: iOS 证书转换相关命令
date: 2019-02-21 15:55:00
tags: 代码库
---

证书有效期：
```
openssl x509 -in xxx.pem -noout -dates
```
生成pem格式的证书：
```
openssl pkcs12 -in CertificateName.p12 -out CertificateName.pem -nodes
```
