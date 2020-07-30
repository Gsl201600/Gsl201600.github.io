---
title: iOS 发布CocoaPods私有库
date: 2019-10-16 16:57:00
tags: 代码库
---

* **需要做的工作包括以下几点**
1. 创建一个本地的仓库，将自己的代码搞进去
2. 将自己的代码上传到远程私有仓库中去
3. 创建一个pods 的描述文件 .podspec
4. 修改.podspec描述文件中的相关的描述信息
5. 创建远程内部私有Spec Repo仓库
6. 向私有的Spec Repo仓库中提交.podspec
7. 在个人项目中的Podfile中增加刚刚制作的好的Pod并使用
8. 后期的升级维护

* **具体详细的步骤如下**
1. 2. 创建远程仓库注意点
* 正规的仓库都有一个license文件，Pods依赖库对这个文件要求比较严格，需要有这个文件，建议使用**MIT**类型的license
* 代码版本要打tag(要在代码版本上传以后打tag)
* pod 支持 .a静态库、.framework 以及文件，不一定要是可运行的工程里面的某个组件
* 放代码的仓库不一定非要是Git仓库，只要是可以获取到相关代码文件就可以，可以是SVN的，也可以是zip包，区别就是在podspec中的source项填写的内容不同

3. 创建一个pods 的描述文件 .podspec
* 如果你已经有先有工程可以使用如下命令直接创建.podspec文件`$ pod spec create MyViewExtension<这个名称一般和创建的项目名称一样就可以>`
* 或者使用如下命令创建完整项目工程目录`$ pod lib create MyViewExtension <这个名称一般和创建的项目名称一样就可以>`使用这个命令会询问如下问题，根据项目情况选择即可
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.10.16.01.png)

4. 修改.podspec描述文件中的相关的描述信息
详情可参考CocoaPods的官网的[PodSpec语法](https://link.jianshu.com/?t=http://guides.cocoapods.org/syntax/podspec.html#group_root_specification)
```
Pod::Spec.new do |s|
  # 项目的名称
  s.name             = "MyViewExtension"   
  # 项目的版本号，通过项目git的tag标签进行对应，这里的标签代表的版本 
  s.version          = "0.0.1"    
  # 项目简单的描述信息        
  s.summary          = "Just Testing."  
  # 项目的详细描述信息，注意，这里的文字的长度，一定要比上面的s.summary长，不然会认为格式不合格
  s.description  = <<-DESC
                      this project provide all kind of KeychainDeviceID for iOS developer 
                   DESC
  # 项目的网页主页信息，这里可以直接写自己的远程仓库的主页的地址
  s.homepage         = "https://github.com/RunOfTheSnail/MyViewExtension"
  # 开源协议
  s.license          = "MIT"   
  # 作者信息          
  s.author             = { "zhangyan" => "17***24@163.com" }                   
  # 这个比较重要，指的就是git的对应的远程仓库的地址以及版本号，版本号直接获取的是上面的s.version
  # 项目地址，这里不支持ssh的地址，验证不通过，只支持HTTP和HTTPS，最好使用HTTPS
  # Supported Keys:
  # :git => :tag, :branch, :commit, :submodules
  # :svn => :folder, :tag, :revision
  # :hg => :revision
  # :http => :flatten, :type, :sha256, :sha1
  s.source       = { :git => "https://github.com/RunOfTheSnail/MyViewExtension.git", :tag => s.version }                      
  # 支持的平台及版本
  s.platform     = :ios, "11.0"
  # 支持的ios最低版本
  s.ios.deployment_target = "7.0"
  # 如果是 Swift 的话指定 Swift 编译版本
  # s.swift_version = "4.0"
  # 必备项，代码源文件地址，如果有多个目录下则用逗号分开,否则"public_header_files"等不可用
  s.source_files  = "GSLXYKeychainDeviceID/KeychainDeviceID/**/*.{h,m}"                               
  # 公开头文件地址
  # s.public_header_files = "Pod/Classes/**/*.h"
  # 所需的系统framework，多个用逗号隔开，不需要后缀名
  # s.framework  = "SomeFramework"
  s.frameworks = "UIKit", "AnotherFramework"
  # 需要弱链接的框架
  # s.weak_framework = "Twitter"
  # s.weak_frameworks = "Twitter", "SafariServices"
  #项目依赖的库文件(这个是系统的库文件),不需要后缀名,比如sqlite,libz等.以lib开头的需要省略掉lib这三个字母.例如:libz需要简写为z否则报错
  # s.library   = "iconv"
  # s.libraries = "iconv", "xml2"
  # 第三方或自己创建的 .Framework的名称
  # s.vendored_frameworks = "YostarLib.framework"
  # 第三方或自己创建的 .a静态库的名称
  # s.vendored_libraries = "libYostarStaticLib.a"
  # 添加资源文件
  # s.resource = "XXX/XXXX/**/*.bundle"
  # s.resources = "XXX/XXXX/**/*.bundle"
  # CocoaPods会把这个库配置成static framework，同时支持Swift和Objective-C
  # s.static_framework = true
  # 依赖关系，该项目所依赖的其他，当在加载的时候也会一块把相关的依赖的库加载下来，如果有多个需要填写多个
  # s.dependency "JSONKit", "~> 1.4" 
  # 是否使用ARC，如果指定具体文件，则具体的文件使用ARC      
  s.requires_arc = true
  # 指定项目配置，如HEADER_SEARCH_PATHS、OTHER_LDFLAGS等
  # s.xcconfig = {"OTHER_LDFLAGS" => "-ObjC"}   
end
```
* 修改完毕之后进行检验一下.podspec的格式有木有问题
```
$ pod lib lint
完整lint格式
$ pod lib lint --allow-warnings --use-libraries --verbose --no-clean --sources='http://10.11.180.29/mobileDevelopers/YZT-Loan-Pod-Spec.git'
--verbose:打印错误
--allow-warnings:允许警告,默认有警告的podspec会验证失败
--fail-fast:遇到错误马上停止，默认会完成全过程再停止
--use-libraries:如果自己私有库包含library,引用了.a、.framework,在验证和提交时需要加
--no-clean:检查问题
--sources:如果依赖了其他不包含在官方specs里的pod，则用它来指明源，比如依赖了某个私有库。多个值以逗号分隔
```

5. 创建远程内部私有Spec Repo仓库
创建远程内部私有Spec Repo仓库, 需要到[Github](https://github.com/)或其他代码托管平台创建远程仓库, 之后将远程仓库克隆到本地，终端执行如下命令:
```
// 这里可以用https或ssh地址方式克隆
$ pod repo add WBSpecs https://github.com/G***00/TestPodspec.git
```
注意：**代码仓库和Spec Repo是需要分开存储的**
克隆成功之后，我们可以查看一下:`$ open ~/.cocoapods/repos`
本地cocoapods目录如下：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.10.16.02.jpg)

6. 向私有的Spec Repo仓库中提交.podspec
* 首先将本地.podspec推送到远程私有repo spec仓库和本地repo spec仓库，终端执行如下命令：`$ pod repo push WBSpecs WBAvoidCrash.podspec 参数解析:repo spec仓库名称 .podspec名称`
* 验证远程是否通过
推送成功之后，终端输入如下命令进行验证`$ pod spec lint WBAvoidCrash.podspec`
* 验证私有仓库是否可用
用pod命令进行搜索，看能否搜索到:`$ pod search WBAvoidCrash`
如果搜索不到，在终端执行如下命令`$ rm ~/Library/Caches/CocoaPods/search_index.json`或者更新本地仓库`$ pod repo update`然后重新search

7. 在个人项目中增加刚刚制作好的Podfile并使用
新建一个测试工程测试，用CocoaPods初始化项目，编辑Podfile文件:
```
#CocoaPods官方spec仓库
source 'https://github.com/CocoaPods/Specs.git'
#自己私有spec仓库
source 'https://github.com/wenmobo/WBSpecs.git'

platform :ios, '8.0'

target 'TestDemo' do
  #防Crash库
  pod 'WBAvoidCrash'
end
```
编辑好podfile文件之后，终端执行:
```
$ pod install 安装时使用，更新库使用update命令
或
$ pod update 更新时使用
```

8. 后期的升级维护
8.1. 更新远程私有库中的代码
8.2. 修改.podspec中的配置，version升级一个版本
8.3. 给当前的远程仓库的代码，重新打个tag，tag和.podspec的version一样
8.4. 远程仓库的代码更新完毕，接下来执行上面的 6.将当前本地的spec文件传到私有Spec Repo仓库的索引库中
8.5. 检查测试一下，有没有上传到私有Spec Repo仓库的索引库中

* **删除私有的Spec Repo`$ pod repo remove [name]`**
其实直接找到以后，手动删除就好了，然后在将Git的变动push到远端仓库即可
* **清理CocoaPods本地缓存**
特殊情况下，由于网络或者别的原因，通过CocoaPods下载的文件可能会有问题

1. 手动删除(`~/Library/Caches/CocoaPods/Pods/Release`目录)

2. 打开终端,输入`$ pod cache list`,会列出所有本地已经缓存的第三方库，在终端中输入`$ pod cache clean AAA`会删除AAA缓存库，使用`$ pod cache clean --all`清除所有缓存
