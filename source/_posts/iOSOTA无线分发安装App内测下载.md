---
title: iOS OTA无线分发安装App内测下载
date: 2019-08-01 14:50:00
tags: 代码库
---

**搭建步骤**
1. 应用`.ipa`文件，可以是`企业级签名`，也可以是`dev签名包`
2. `manifest.plist`文件，`plist`文件和`ipa`文件必须放在支持`https://`服务器上，而且必须是公网`ssl`，自签名及免费的`https`不可用（本文以`GitHub`为例）
3. 下载应用的`html`页面
*****************************
* `manifest.plist`内容如下
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>items</key>
    <array>
        <dict>
            <key>assets</key>
            <array>
                <dict>
                    <key>kind</key>
                    <string>software-package</string>
                    <key>url</key>
                    <string>https://raw.githubusercontent.com/***/testIPA/master/Unity-iPhone.ipa</string>
                </dict>
            </array>
            <key>metadata</key>
            <dict>
                <key>bundle-identifier</key>
                <string>com.*****EN.****</string>
                <key>bundle-version</key>
                <string>0.1</string>
                <key>kind</key>
                <string>software</string>
                <key>title</key>
                <string>应用名称</string>
            </dict>
        </dict>
    </array>
</dict>
</plist>
```
* 将应用的`.ipa`文件，`manifest.plist`文件以及`html`下载页面，上传到`GitHub`上（支持`https://`服务器上）
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.01.01.png)

* **注意：获取文件路径正确姿势**

![获取.ipa和图片下载路径，他们的获取方式一样](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.01.02.png)
![获取manifest.plist文件路径](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.01.03.png)
![点击Raw按钮后，会进入以下页面](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.01.04.png)

* 修改`manifest.plist`文件，将获取的下载路径，填写到`manifest.plist`文件中对应位置
* `html`简易下载页面，下载链接必须是这样的格式`itms-services://?action=download-manifest&url=一个plist文件的地址`
```
<!DOCTYPE HTML>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <title>Install</title>
    </head>
    <body>
        <p align=center>
          <font size="10">
            <a style="color:#69DEDA" href="itms-services://?action=download-manifest&url=https://raw.githubusercontent.com/***/testIPA/master/DownloadPlist.plist">点击安装</a>
          </font>
        </p>
    </body>
</html>
```
* **附：如何直接在`github`上预览`html`网页效果**
1. 将`github`上`demo`的`html`文件链接复制到，打开下面网址后出现的输入栏中，点击按钮即可。
```
http://htmlpreview.github.io/
```
[GitHub & BitBucket HTML Preview](http://htmlpreview.github.io/)
2. 在`HTML`文件的地址前面加上`htmlpreview.github.io/?`

```
htmlpreview.github.io/?文件地址
```
3. 在`github`上`demo`的仓库页面，点击`setting`按钮，找到`GitHub Pages`版块，选择`Source`为`master branch`，然后保存（或者在`Theme Chooser`处，点击`Change theme`，进入页面后选择主题保存，这样也可以）页面会出现一个新链接，在链接后面加上`demo`的`HTML`文件名，即可跳转到`demo`显示的页面
* **附：Github如何上传超过100M的大文件**(使用 [Git LFS](https://github.com/git-lfs/git-lfs))
使用方式：先安装`Git LFS`的客户端，然后在将要`push`的仓库里重新打开一个`bash`命令行：
1. 只需设置1次`LFS`：`git lfs install`
2. 然后跟踪一下你要`push`的大文件的文件或指定文件类型`git lfs track "*.pdf"`当然还可以直接编辑`.gitattributes`文件
3. 以上已经设置完毕，其余的工作就是按照正常的`add, commit, push`流程就可以了
**注：**`sourcetree`集成的有`LFS`使用也非常方便
* **附：使用的SourceTree提交到git远程仓库的时候时出现了问题，一直处于这种状态`POST Git-receive-pack (chunked)`**
* 解决办法
打开`SourceTree`右上角的，`设置 -> 高级 -> 编辑配置文件`，来打开配置文件，在配置文件中添加如下配置，最后保存，重新尝试推送到仓库就可以了
```
[http] 
      postBuffer = 524288000
```
