---
title: Mac OS + Mac PE + Win PE 三合一 U盘制作教程
date: 2020-04-08 15:59:00
tags: 随笔
---

开始之前需要准备一下工具：
* `移动硬盘`或者`U盘`一个
* `Mac OS`原版安装文件
* `Mac PE`
* `Win PE`
* `DiskGenius`分区工具

1. `Win PE`制作
下载好`U盘魔术师V5全能版`或者`通用PE工具箱`等`Win PE`制作软件，安装到电脑打开，然后插入`U盘`；一般保持默认设置就行，`Win PE`制作完成。

2. `Mac OS`分区制作
打开`DiskGenius`分区工具，找到刚刚制作好的`U盘`，然后选中这个`U盘分区`，右击菜单选中`调整分区大小`，如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.01.png)

打开调整窗口，把鼠标放在分区的右边，出现拖拉箭头，然后往左拉，或者在下方直接填写你要分的空间大小，如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.02.png)

`Mac OS`分区的空间一般`8.5G`，也可以设置更大，看个人。然后点击`开始`，弹出对话框，选中`是`；制作完成之后点击`完成`，就会出现空闲`8.5G`，如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.03.png)

右击空闲的分区，选择`建立新分区`，选中`NTFS`格式，`4K对齐`，如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.04.png)

选择`确定`，然后点击左上角的`保存更改`，提示你格式化分区，选择`是`，格式化完成之后，`8.5G`的`Mac OS`分区就制作好了。

3. 制作`Mac PE`分区，分`9G`以上；方法跟制作`Mac OS`分区是一样的，这里不再重复，如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.05.png)

分区基本做好了，现在转到苹果系统去写入系统文件。

4. 格式化两个`Mac`分区

打开苹果电脑的`磁盘工具`，找到刚刚分出来的`9G`容量的分区，然后选择`抹掉`，名称为`Mac PE`，只为做区分用，可以随意命名；格式为`Mac OS扩展 日志式`；然后选择`8.5G`的`Mac OS`安装分区，步骤和上面一样，如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.06.png)

5. `Mac PE`系统文件写入，打开下载好的`iFen.OS X PE`，解压的过程选择`跳过`，不跳过也行，目的是为了加载进磁盘工具里，回到`磁盘工具`的界面，会出现解压出来的镜像，然后选中`Mac PE`分区，选择菜单的`恢复`按钮，恢复来源选择刚刚解压出来的`PE镜像`，点击`恢复`，如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.07.png)

**注意：**`iFen.OS X PE`是基于`Mac OS 10.14`制作的，所以要用`Mac OS 10.14`的电脑才能恢复，否则会恢复失败，恢复过程的快慢就要看`U盘`的速度了，大概`5分钟`的时间。

6. `Mac OS`系统文件写入，以`Mac OS 10.14`系统为例，不同的系统制作代码不同，代码中的`MyVolume`为上面命名的`U盘名称`，下载好`Mac OS 10.14.1`系统，并把它放在`Mac的应用程序`里备用，打开`终端`，在终端输入代码：`sudo /Applications/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume`
等待制作完成，大概`5分钟`，完成之后；如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.08.png)

至此，整个三合一的启动盘制作完成了，三合一U盘电脑启动界面，如图：
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img2020/2020.04.08.09.png)

[附：Mac OS X + Mac PE + Win PE 三合一 U盘制作教程](https://www.applex.net/threads/mac-os-x-mac-pe-win-pe-u.92345)
[附：[官方文档] 如何创建可引导的 macOS 安装器](https://support.apple.com/zh-cn/HT201372)
[附：Mac PE下载地址 密码:6jkp](https://pan.baidu.com/s/1mGgc50GOZAaVsYbbHAjyHw)
[附：最新版Mac PE](https://www.applex.net/threads/mac-pe19.93480)
