---
title: iOS Xcode 添加调试真机设备和模拟器
date: 2019-03-05 15:02:00
tags: 代码库
---

1. 高版本`Xcode`调试低版本真机设备
* 前往文件夹或者找到`Xcode`安装包右键显示包内容查找路径
```
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport
```

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.05.01.png)

最新的`Xcode`默认是没有`7.0`和`7.1`文件夹，我们可以从`Xcode 7`的`DeviceSupport`文件夹下拷贝出来，然后复制进去

* 前往文件夹或者找到`Xcode`安装包右键显示包内容查找路径
```
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/SDKSettings.plist
```
查找到`SDKSettings.plist`文件，在`DEPLOYMENT_TARGET_SUGGESTED_VALUES`字段下面添加`7.0`和`7.1`，如果第一步添加了更老的版本，这里也一起添加了
![SDKSettingPlist](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.05.02.jpg)

如果是`Xcode 8`以下的版本调试适配`iOS 10`，方法是一样的，只不过需要在高版本的`Xcode`里面把配置文件拷贝出来

如果`SDKSettings.plist`这个文件提示无法修改的话，可以先将这个文件拷贝一份到桌面，修改后再覆盖进去即可

2. `Xcode`添加模拟器
* 首先打开`Xcode`，找到`Add Additional Simulators`点击
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.05.03.png)

* 点击`+`号选择`Add Simulator`
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.05.04.png)

* 这里选择你需要的模拟器`Type`和`Version`后点击`Create`就可以了
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.03.05.05.png)
