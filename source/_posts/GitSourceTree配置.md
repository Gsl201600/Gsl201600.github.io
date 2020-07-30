---
title: Git SourceTree配置
date: 2019-02-26 15:48:00
tags: 代码库
---

创建SSH Key，因为本地的Git仓库与Github远程仓库之间是通过SSH加密的。首先，需要到主目录上查看是否有.ssh目录，再查看.ssh目录下有没有id_rsa和id_rsa.pub文件，如下
![查找.ssh目录](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2018.06.27.01.jpg)

发现没有上述的两个文件，这时需要创建：
```
ssh-keygen -t rsa -C "youremail@example.com"
```

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2018.06.27.02.png)

出现上述描述，就证明你成功了，然后到主目录下找到.ssh目录，查看id_rsa和id_rsa.pub文件，id_rsa是私钥，需要自己保留好，id_rsa.pub是公钥，别人知道也无妨。
登录Github账户，打开Account settings，SSH Keys页面,添加id_rsa.pub文件的内容。
