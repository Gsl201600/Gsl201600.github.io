---
title: CocoaPods安装与使用步骤详解
date: 2019-07-17 16:00:00
tags: 代码库
---

> **目录**
> * CocoaPods安装过程
> * CocoaPods的使用
> * 删除cocoapods已导入项目的第三方库和移除项目中的cocoapods

* **CocoaPods安装过程**
1. 安装并载入rvm环境
* 打开终端，输入指令`$ rvm -v`
![rvm 环境检测](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.07.17.01.png)

* 安装rvm
安装指令是`$ curl -L https://get.rvm.io | bash -s stable`
载入RVM环境：`$ source ~/.rvm/scripts/rvm`
检查是否安装成功：`$ rvm -v`
![安装rvm](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.07.17.02.png)

* rvm命令安装Ruby环境
查看当前ruby版本`$ ruby -v`（检查当前版本,当ruby版本低于2.2.2时，安装cocoapods会报错）
查看所有ruby版本`$ rvm list known`
`$ rvm list known`命令会查询所有的ruby版本，找到最高版本号进行安装；若版本库里没有最新版本，输入：`$ rvm get head`升级到最新的存储库源版本
安装指定版本，输入指令：`$ rvm install 2.5.1` (选择较高版本)
等待漫长的下载，编译过程，完成以后，Ruby, Ruby Gems 就自动安装好了
**注意：如果安装失败请手动安装 Homebrew `$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`再执行该`$ rvm install 2.5.1`命令，查询Homebrew是否安装成功的命令`$ brew -v`**
查看已经安装的ruby版本`$ rvm list`
![查看已安装的ruby版本](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.07.17.03.png)

卸载一个已安装版本`$ rvm remove 2.5.1`
2. 设置默认Ruby版本
安装好rvm之后可以指定特定Ruby版本为系统默认版本，输入命令：`$ rvm 2.5.1 --default`也可以指定其他版本号，前提是有用rvm install 安装过那个版本
3. 检查更新ruby版本环境
cocoapods是用gem ruby实现的，想要使用它首先需要有gem ruby的环境。且Mac的macos系统默认已经可以运行ruby。建议gem ruby包环境升级到2.6.x以上
* 检查gem ruby版本号：`$ ruby -v` `$ gem -v`
Gem是管理Ruby库和程序的标准包，如果它的版本过低也可能导致安装失败，解决的办法是更新gem版本
* 更新gem ruby版本`$ gem update --system`
4. 检查ruby源并移除
* 检查ruby源`$ gem sources -l`
因为Ruby环境默认的的软件源`rubygems.org`被屏蔽了，国内那面永远需要翻越的墙，我们需要来修改更换源，把源切换至ruby-china
* 移除掉原有的源`$ gem sources --remove https://rubygems.org/`
* 添加国内最新的源`$ gem sources -a https://gems.ruby-china.com`
* 检查是否添加成功`$ gem sources -l`
到这里就已经把Ruby环境安装成功
5. 安装CocoaPods
`$ gem install -n /usr/local/bin cocoapods`
6. 查看是否安装成功并更新
* 查看是否成功`$ pod --version`
* 更新Podspec索引文件，创建本地索引库,如果没有报错，就说明一切安装成功了；这个过程需要一些时间`$ pod setup`
查看本地索引库:`$ open ~/.cocoapods/repos`
* **CocoaPods的使用**
1. 用Xcode创建一个工程，并创建podfile配置文件
* 进入项目目录`$ cd ~`
* 创建Podfile文件`$ touch Podfile`
* 打开编辑，使用`$ vi Podfile`输入i进入编辑，编辑完成后按 esc 然后输入`:wq`按回车键 ，保存并退出
* 编辑Podfile文件
我们可以在Podfile文件中写入需要用到的第三方库按如下格式：
```
platform :ios, '9.0'
use_frameworks!
target 'TestDemo' do
pod 'Alamofire', '~> 4.0.1'
pod 'Kingfisher', '~> 3.1.1'
end
```
Swift的pod文件在于use_frameworks! 这一句是必须的，作用是把三方库打包成静态库，而oc是不需要的
2. 安装依赖库
`$ pod install` (后续添加框架可直接`pod update`)
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.07.17.04.png)

* **删除cocoapods已导入项目的第三方库和移除项目中的cocoapods**
1. 删除项目中已经由cocoapods配置好的第三方
* 打开项目中的Podfile文件
* 删除选中的pod Snapkit的命令行
* 打开终端cd到当前项目的根目录下重新执行`$ pod install --verbose --no-repo-update`或者直接`$ pod install`
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.07.17.05.png)

2. 删除项目中的cocoapods
* 手动删除
> 1. 删除本地文件(Podfile、Podfile.lock、Pods文件夹、xcworkspace文件)
> 2. 打开项目，在Frameworks文件夹下，删除Pods.xcconfig和libPods.a
> 3. 进入项目Build Phases，删除Copy Pods Resources、Embed Pods Frameworks和Check Pods Manifest.lock 三项
> 4. 删除了CocoaPod管理的第三方代码，在工程里面引用的第三方代码都会报错，需要删除对应的代码
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.07.17.06.jpg)

* 通过第三方插件删除
> 1. 安装cocoapods-deintegrate命令：`$ sudo gem install cocoapods-deintegrate`
> 2. 然后到工程目录下面执行命令：`$ pod deintegrate`
> 3. 手动删除.xcworkspace，libPods.a，Podfile，Podfile.lock文件
> 4. 如果想要重装的话保留Podfile，再执行命令：`$ pod install`就好了

[附：最新CocoaPods安装与使用步骤详解](https://www.jianshu.com/p/d298c21fc95a)
[附：RubyGems 镜像](https://gems.ruby-china.com)
