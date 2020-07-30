---
title: iOS Crash文件获取及符号化
date: 2020-04-29 15:46:00
tags: 代码库
---

1. `Crash`文件获取

* 大致可以分为两种方式：远程获取和本地获取；具体可以分为如下四种途径

1.1. 远程获取；已经上传到`iTunes Connect`的应用，可以通过`iTunes Connect`的`App分析`查看`App`崩溃情况`不会有崩溃日志`，如果是`TestFlight`测试，则可以在`iTunes Connect`获取到崩溃日志

1.2. 远程获取；通过`Xcode`菜单`Window -> Organizer -> Crashes`获取用户的崩溃日志
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.29.01.png)

* 注意：以上两种途径都需要登录开发者账号，并且需要用户共享`iPhone`分析，才能够获取到用户的崩溃日志
* 注意：官方提供的崩溃信息并不是实时的，只能查看两天之前的崩溃信息，需要实时的可以使用第三方工具

1.3. 本地获取；在手机上`设置 -> 隐私 -> 分析与改进 -> 分析数据`中，根据应用名称和日期时间找到你需要的日志，点击进去后，右上角会有个分享按钮，分享给`Mac`

1.4. 把手机连接到`Mac`，通过`Xcode`菜单`Window -> Devices and Simulators -> Devices -> View Device Logs`获取用户的崩溃日志

* 注意某些`iOS`系统会没有上面提到的分享按钮，这时候可以全选复制，再发送给`Mac`
![1.1该图为早期iTunes Connect官网，具体以当前官网为主](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.29.02.png)

2. `Crash`文件符号化

* 大致也是分为两种方式：使用`Xcode`自动符号化和通过手动命令行工具`symbolicatecrash`符号化；这两种方式原理一样，都需要`dSYM`文件，只不过前者是`Xcode`自动帮我们完成的
* 注意：如果你们的应用是通过`Xcode`上传`iTunes Connect`的，并同时上传了`.xcarchive`文件`(实际上是一个文件夹，包含.ipa和.dSYM文件)`，`Xcode`会默认帮你勾选该选项，那么从`iTunes Connect`获取到的日志就已经是符号化过的了

2.1. 使用`Xcode`自动符号化`Crash`文件，`Xcode`自带的工具非常好用

* 如果你用的`Mac`就是打包的机子，并且得到了发生崩溃的手机，那么手机连接电脑，通过`Xcode`菜单`Window -> Devices and Simulators -> Devices -> View Device Logs`找到自己的日志，就是符号化过后的，如果没有符号化，就稍微等待一会儿，或者右击点出菜单选择`Re-Symbolicate Log`
* 如果只有`Mac`出包机，没有手机只有崩溃日志，那么同样可以通过`Xcode`菜单`Window -> Devices and Simulators -> Devices -> View Device Logs`把崩溃日志直接拖进去，就是符号化过后的，如果没有符号化，就稍微等待一会儿，或者右击点出菜单选择`Re-Symbolicate Log`
* 注意：在有些版本的`Xcode`是拖不进去的，遇到这种情况可以用下面的手动符号化方式
* 注意：上面的方法不一定要是出包机，本质是只要你的电脑上有`dSYM`文件，`Xcode`就能自动找到他并为你符号化
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.29.03.png)

2.2. 通过终端命令行工具`symbolicatecrash`符号化
大概需要如下三个文件，下面是获取这些文件的方法
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.29.04.png)

* 通过菜单`Xcode -> Window -> Organizer -> Archiver`找到打包的项目，右键`Show In Finder`，找到`AppName.xcarchive`，右键显示包内容，找到`AppName.app.dSYM`
* 在桌面创建一个文件夹`tmp`，将以上两个文件拷贝到`tmp`文件夹中
* 打开终端，用`find /Applications/Xcode.app -name symbolicatecrash -type f`查找`symbolicatecrash`，其中`/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash`路径是需要的`symbolicatecrash`文件，将`symbolicatecrash`文件也拷贝到`tmp`文件夹中
* 将需要分析的`crash`文件也拷贝到`tmp`文件夹中，`crash`文件的格式可能是`.beta`、`.crash`或`.ips`
* 在终端中使用以下命令行，`crash`文件格式以`.crash`为例
```
# 进入到 tmp 文件夹中
cd ~/Desktop/tmp 

# 分析 crash 文件，会在 `tmp` 文件夹中生成 crash.log 文件
./symbolicatecrash ./xxx.crash ./AppName.app.dSYM > crash.log
或./symbolicatecrash ./xxx.crash ./.app.dSYM > crash.log
```
* 如果终端报类似这样的错`zsh: permission denied: ./symbolicatecrash`
说明是`symbolicatecrash`文件有问题，可能该文件不是本机获取的，或者是之前获取的、`Xcode`升级等问题造成的，重新在本机上按上面方法获取即可
* 如果终端报类似这样的错`Error: "DEVELOPER_DIR" is not defined at ./symbolicatecrash line 69.`
尝试以下命令后，再重复上面命令，正常情况就可以分析bug了`export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer`

3. 符号化之前，首先得确保`Crash`文件和`dSYM`这两个文件里面的`UUID`是一致的，如果不一致，就说明不是本次`Crash`对应的文件，就不能进行符号化；查看`dSYM`文件里面的`UUID`命令：`dwarfdump --uuid AppName.app.dSYM`；查看`Crash`文件文件的`UUID`就比较简单了，直接打开，`Crash`最上面的就是各种信息，不同系统版本给的格式可能会有不同，下图内容为参考
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.29.05.png)

* `Xcode`在`Debug`模式下默认关闭生成`dSYM`文件，`Release`模式下默认生成`dSYM`文件的， 要生成`dSYM`文件需要查看一下项目的`Build Settigns -> Build Options -> Debug information Format`属性；只有该属性设置为`DWARF with dSYM File`时，编译才会生成`dSYM`文件
* 该文是在`Xcode 11.2`和`iOS 13.2`上写的教程，不同的系统版本的`Xcode`和手机系统获取路径和符号化方式会有变化
