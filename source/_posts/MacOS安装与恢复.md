---
title: Mac OS 安装与恢复
date: 2019-10-09 16:31:00
tags: 随笔
---

1. 通过 macOS 恢复功能启动
要通过 macOS 恢复功能启动，请开启 Mac 并立即按住键盘上的以下组合键之一。通常建议您使用 Command-R-电源键
* Command (⌘)-R-电源键
安装您的 Mac 上装有的最新 macOS
* Option-⌘-R-电源键
升级到与您的 Mac 兼容的最新 macOS
* Shift-Option-⌘-R-电源键
安装 Mac 随附的 macOS 或与它最接近且仍在提供的版本
当您看到 Apple 标志、旋转的地球的提示时，请松开这些按键。当您看到“实用工具”窗口时，即表示您已通过 macOS 恢复功能启动
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.10.09.01.png)

* 如果您在安装 macOS 之前需要抹掉磁盘，请从“实用工具”窗口中选择“磁盘工具”，然后点按“继续”。除非您要出售或赠送您的 Mac，或者遇到一个需要您抹掉磁盘的问题，否则您可能不需要抹掉磁盘
* 从“实用工具”窗口中选取“重新安装 macOS”（或“重新安装 OS X”）点按“继续”，然后按照屏幕上的说明来选取磁盘并开始安装

2. 制作启动盘来恢复安装
* 首先准备一个8GB或更大容量的U盘，下载好macOS Mojave正式版的安装程序，并把它放在Mac的「应用程序」里备用
* 打开 “应用程序 → 实用工具 → 磁盘工具”，将U盘「抹掉」(格式化) 成「Mac OS X 扩展（日志式）」格式、(GUID 分区图可选项)，并将 U 盘命名为「Mojave」(下图序号3处)。注意：这个盘符名称必须与后面的命令里的名称一致，需要认真看清楚
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.10.09.02.png)

* 在“终端”中键入或粘贴以下命令之一。这些命令假设安装器仍位于您的“应用程序”文件夹中，并且 MyVolume 是 USB 闪存驱动器或您正在使用的其他宗卷的名称。如果不是这个名称，请相应地替换为 MyVolume
* 制作 macOS Mojave 启动盘，U盘名称(必须与下面命令对应)，然后拷贝这段命令：`sudo /Applications/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume`
* 制作 macOS High Sierra 启动盘，U盘名称(要与下面命令对应)，拷贝这段命令：`sudo /Applications/Install\ macOS\ High\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume`
* 制作旧版本的 macOS Sierra，拷贝这段命令：`sudo /Applications/Install\ macOS\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume --applicationpath /Applications/Install\ macOS\ Sierra.app`
* 制作El Capitan，拷贝这段命令：`sudo /Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app`
* 当“终端”显示这个操作已完成时，该宗卷的名称将与您下载的安装器名称相同，例如“Install macOS Mojave”。您现在可以退出“终端”并弹出宗卷
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.10.09.03.png)

* 当你制作好 macOS Mojave 的安装盘 U 盘之后，你就可以利用它来给Mac电脑格式化重装 (抹盘安装)了。操作的方法非常简单：按下电源键开机，按住 Option 键不放，直到出现启动菜单选项，这时选择安装U盘 (黄色图标) 并回车，就可以开始安装了，在过程中你可以直接覆盖安装系统(升级)，也可以通过“磁盘工具”对 Mac 的磁盘式化或者重新分区等操作实现全新干净的安装。之后就是一步一步的安装直到完成了

[附：制作 macOS Mojave U盘USB启动安装盘方法教程 (全新安装 Mac 系统)](https://www.iplaysoft.com/macos-usb-install-drive.html)
[附：[官方文档] 如何通过 macOS 恢复功能重新安装 macOS](https://support.apple.com/zh-cn/HT204904)
[附：[官方文档] 如何创建可引导的 macOS 安装器](https://support.apple.com/zh-cn/HT201372)
