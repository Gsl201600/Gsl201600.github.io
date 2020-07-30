---
title: iOS 使用Jenkins持续集成(简称CI)
date: 2019-11-27 16:56:00
tags: 代码库
---

1. 安装`jenkins`
1.1. 直接到[官网](https://link.jianshu.com/?t=http://jenkins-ci.org/)下载安装包，通过安装包安装
1.2. 通过`Homebrew`使用命令行安装
```
1. 安装Homebrew
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
2. 安装Jenkins
$ brew install jenkins
3、启动Jenkins
$ jenkins
```
* `jenkins`需要`java`环境，如果没有安装会有提示，[java安装地址](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.oracle.com%2Ftechnetwork%2Fjava%2Fjavase%2Fdownloads%2Fjdk8-downloads-2133151.html)

一切顺利的话，打开浏览器输入：`http://localhost:8080/`就能看到`jenkins`已经运行起来了，如果你更换了端口就是你后来设置的端口。接下来打开`Jenkins`后会让去一个填写`password`的页面如下图，存储`password`的地方就是图片上那行红色字体目录
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.01.png)

然后将我们得到的`password`输入到`Administrator password`中，即可进入如下界面，接着安装一些建议的插件`左边的`，安装过程中，有的插件可能会安装失败，强烈建议点击右下角的重试，直到把建议安装的都装好
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.02.png)

插件安装完成后，可能不会自动跳转，刷新下界面即可，在刷新后的界面中注册，输入用户名和密码，建议输入后点蓝色按钮保存完成

2. 安装`jenkins`插件
如果要使用`Jenkins`的插件构建工程的，需要在开始新建工程前安装一些`Jenkins`插件，在可选插件中选择我们需要的插件进行安装
```
1. Xcode integration
2. GIT plugin
3. GitLab Plugin
4. Gitlab Hook Plugin
5. Keychains and Provisioning Profiles Management
```
我们今天使用`Execute shell` `Shell脚本`构建工程

3. `jenkins`的使用
3.1. 构建一个自由风格的软件项目
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.03.png)

3.2. `General`参数
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.04.JPG)
可以设置包的保留天数和最大保留个数，这些可以根据需要进行调整，可以不要选
* `jenkins`插件配置多个项目`extended choice parameter`插件主要是构建的时候可以多选框来选择要构建的项目模块

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.05.png)
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.06.png)
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.07.png)
![($+上面的Name)就可以获取该值](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.08.png)

3.3. 源码管理
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.09.png)
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.10.png)

3.4. 构建触发器设置
触发器可自定义的地方很多，可以根据项目需要选择`可省略`
* 定时构建：不管`SVN`或`Git`中数据有无变化，均执行定时化的构建任务
* 轮询`SCM`：只要`SVN`或`Git`中数据有更新，则执行构建任务
日程表的填写内容有`5`个参数，从左到右的参数含义如下：
⦁ 第`1`个参数：分钟`minute`，取值`0~59`
⦁ 第`2`个参数：小时`hour`，取值`0~23`
⦁ 第`3`个参数：天`day`，取值`1~31`
⦁ 第`4`个参数：月`month`，取值`1~12`
⦁ 第`5`个参数：星期`week`，取值`0~7`，`0`和`7`都是表示星期天
`5`个参数可选择性设定，不写死的参数用`*`号代替，参数之间用空格隔开。例如：
```
"0 21 * * *"表示每晚21点0分自动化构建一次
"0 * * * *"表示每个小时的第0分钟执行一次构建
"H/5 * * * *"每隔5分钟构建一次
"H H/2 * * *"每两小时构建一次
"H H 30 * *"每月30号构建一次
"H(0-29)/10 * * * *"每个小时的前半个小时内的每10分钟
"0 8-17/2 * * 1-5"周一到周五，8点~17点，两小时构建一次
"H H 1,15 1-11 *"每月1号、15号各构建一次，除12月等
```
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.11.png)

3.5. 构建环境设置
本文使用的是`shell`脚本构建工程，所以该项可以省去
3.6. 构建
有两种方式打包，一是用`Xcode`插件打包，二是用`Shell`脚本打包，本文选择第二种
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.12.png)

* `iOS`自动打包—`Jenkins Shell`如下：
```
## !/bin/sh
## 项目名
TARGET_NAME=NNAlgorithm
## Scheme名
SCHEME=NNAlgorithm
##=======================
## 编译类型
BUILD_TYPE=Release
## 当前目录
SORCEPATH=${WORKSPACE}
## workspace名
SPACE=${WORKSPACE}/${TARGET_NAME}.xcodeproj
##xcarchive文件的存放路径
ARCHIVEPATH=$SORCEPATH/build/$SCHEME.xcarchive
## ipa文件的存放路径
EXPORTPATH=$SORCEPATH/build/$SCHEME
## ExportOptions.plist文件的存放路径，该文件要存放在这个路径下内容如下
EXPORTOPTIONSPLIST=$SORCEPATH/build/ExportOptions.plist
## 导出后的ipa路径
EXPORTPATHIPA=$SORCEPATH/build/$SCHEME/$SCHEME.ipa

echo -e "============First Build Clean============"
## 清理缓存
## 如果工程使用的是cocoapods，则'-project %s.xcodeproj'替换为'-workspace %s.xcworkspace'
xcodebuild clean -project $SPACE -scheme ${SCHEME} -configuration ${BUILD_TYPE}
echo -e "============Build Clean============"
## 输出关键信息
echo -e "  TARGET_NAME    : ${TARGET_NAME}"
echo -e "  BUILD_TYPE    : ${BUILD_TYPE}"
echo -e "  SORCEPATH    : ${SORCEPATH}"
echo -e "  ARCHIVEPATH    : ${ARCHIVEPATH}"
echo -e "  EXPORTPATH    : ${EXPORTPATH}"
echo -e "  EXPORTOPTIONSPLIST    : ${EXPORTOPTIONSPLIST}"
echo -e "============Build Archive============"

## 导出archive包
xcodebuild archive -project ${SPACE} -scheme ${SCHEME} -archivePath $ARCHIVEPATH
echo -e "============Build Archive Success============"

echo -e "============Export IPA============"
## 导出IPA包
xcodebuild -exportArchive -archivePath $ARCHIVEPATH -exportPath ${EXPORTPATH} -exportOptionsPlist ${EXPORTOPTIONSPLIST}
echo -e "============Export IPA SUCCESS============"

## 编译完成时间 20181030_0931
BUILD_DATE="$(date +'%Y%m%d_%H%M')"

## info.plist路径
PROJECT_INFOPLIST_PATH="${SORCEPATH}/${TARGET_NAME}/Info.plist"
## 取版本号
BUNDLESHORTVERSION=$(/usr/libexec/PlistBuddy -c "print CFBundleShortVersionString" "${PROJECT_INFOPLIST_PATH}")
## 取build值
VERSION=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" "${PROJECT_INFOPLIST_PATH}")
## ipa更名规则  项目名V版本_年月日_时分
IPANAME="${TARGET_NAME}V${BUNDLESHORTVERSION}_${BUILD_DATE}.ipa"
## 更名后ipa路径
EXPORTPATHNEWIPA=$EXPORTPATH/$IPANAME

echo -e "============Export end :${BUILD_DATE}============"
echo -e "============IPA Old Name: ${EXPORTPATHIPA}============"
echo -e "============IPA New Name: ${EXPORTPATHNEWIPA}============"

## IPA更名
cp $EXPORTPATHIPA $EXPORTPATHNEWIPA
echo -e "============Create New Name Success============"
## 删除老IPA
rm $EXPORTPATHIPA
echo -e "============Delete Old Name Success============"

#userKey和apiKey需要在蒲公英的账号设置中查找
userKey="xxx"
apiKey="xxx"
#蒲公英打包
curl -F "file=@${EXPORTPATHNEWIPA}" \
-F "uKey=${userKey}" \
-F "_api_key=${apiKey}" \
-F "isPublishToPublic=2" \
http://www.pgyer.com/apiv1/app/upload
```
* `ExportOptions.plist`
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>provisioningProfiles</key>
    <dict>
        <key>com.Y***ane</key>
        <string>azur***_dev</string>
    </dict>
    <key>method</key>
    <string>development</string>
    <key>signingCertificate</key>
    <string>iPhone Developer</string>
    <key>signingStyle</key>
    <string>manual</string>
    <key>teamID</key>
    <string>42***ZL</string>
    <key>compileBitcode</key>
    <false/>
    <key>uploadSymbols</key>
    <false/>
</dict>
</plist>
```
* 其中`plist`文件中的`method`参数有如下几个方法：`app-store, ad-hoc, enterprise, development`
3.7. 构建后操作
* 邮件通知系统，通过`系统管理`→`系统设置`，进行邮件配置
* 设置`jenkins`地址和管理员邮箱地址
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.13.JPG)

* 设置发件人等信息
这里的发件人邮箱地址切记要和系统管理员邮件地址保持一致
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.14.png)

* **注：上图的Password为邮箱的SMTP授权秘钥，至此系统管理处的内容已配置完成**
* 配置`Jenkins`自带的邮件功能(测试邮件功能是否正常使用，可以不配置，不影响)
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.15.png)

和上面`Extended E-mail Notification`配置一样即可，点击`Test configuration`，收到邮件并且显示`Email was successfully sent`，代表邮件配置成功，接下来可以去项目中具体配置就可以使用了

* 进入项目，然后找到构建后操作，点击`增加构建后的操作步骤`，点击`Editable Email Notification`
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.16.png)
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.17.png)

* 至此所有的配置已完成，点击应用后保存，`enjoy it！`

```
Project Recipient List：这个项目的需要发送邮件给哪些人，可以在这里输入多个邮箱，中间以英文逗号隔开
Project Reply-To List：保持默认即可，这个是收到邮件的人回复邮件时候回复给谁用的，一般不会回复邮件
Content Type：可以选择Html或者Default也行，因为我们在jenkins系统设置中的默认格式就是html
Default Subject： 邮件主题，可以书写成：XXX项目iOS打包通知:$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS! 分析下这几个参数什么意思：$PROJECT_NAME 构建项目的名称；$BUILD_NUMBER 构建的号码；$BUILD_STATUS 构建状态，这几个参数，它会自动读取，按照这种格式书写即可
Default Content：邮件内容，以下内容为模板，可直接复制修改使用：

<hr/>
本邮件是程序自动下发的，请勿回复！<br/><hr/>
项目名称：$PROJECT_NAME<br/><hr/>
构建编号：$BUILD_NUMBER<br/><hr/>
构建状态：$BUILD_STATUS<br/><hr/>
触发原因：${CAUSE}<br/><hr/>
构建日志地址：<a href="${BUILD_URL}console">${BUILD_URL}console/</a><br/><hr/>
构建地址：<a href="$BUILD_URL">$BUILD_URL</a><br/><hr/>
构建报告：<a href="${BUILD_URL}testReport">${BUILD_URL}testReport/</a><br/><hr/>
变更集:${JELLY_SCRIPT,template="html"}<br/><hr/>
```
4. [Jenkins卸载方法（Windows/Linux/MacOS）](https://www.cnblogs.com/EasonJim/p/6277708.html)
* 如果使用`brew`安装的，可以执行以下命令`$ brew uninstall jenkins`
* **注：Jenkins修改工程的工作空间**
在项目的配置中点击高级，选择使用自定义工作空间，输入工作空间路径即可
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.18.png)
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.11.27.19.png)
