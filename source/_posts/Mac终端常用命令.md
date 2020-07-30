---
title: Mac 终端常用命令
date: 2018-12-25 18:30:00
tags: 代码库
---

**目录操作**

|命令|功能描述|示例|
|---|---|---|
|pwd|显示当前路径|pwd|
|cd /|跳转到根路径下|cd /|
|cd ..|跳转到上级路径|cd ..|
|cd  或 ~|跳转到当前登录用户的家目录|cd  或 cd ~|
|ls|可以列出当前路径下的所有可见文件和文件夹|ls|
|ls -a|列出当前路径下的所有文件和文件夹|ls -a|
|mkdir|在当前路径下新建一个文件夹|mkdir new_dir|
|mkdir|新建多个文件夹|mkdir dir1 dir2 dir3|
|rm -r|删除一个文件夹|rm -r dir|
|rm -r|删除多个文件夹|rm -r dir1 dir2 dir3|

**文件操作**

|命令|功能描述|示例|
|---|---|---|
|touch|在当前路径下新建一个文件|touch file_name|
|rm|删除一个文件|rm file_name|
|rm|删除多个文件|rm file1 file2 file3|
|cat|显示或连接文件|cat /路径/libiPhone-lib.a|
|strings|在对象文件或者二进制文件中查找可打印的字符串，与grep配合使用，输入strings -h查看strings命令的用法|strings /路径/libiPhone-lib.a \| grep "UIWebView"|

**其他命令**

|命令|功能描述|示例|
|---|---|---|
|date|显示系统的当前日期和时间|date|
|cal|显示日历|cal|
|clear 或 Ctrl + L|清除屏幕或窗口内容|clear 或 Ctrl + L|
|env|显示当前所有设置过的环境变量|env|
|say|朗读一段文字|say guan|
|curl|Mac 查看本机公网IP 命令|curl ipinfo.io/json|
|curl|Mac 查看本机公网IP 命令|curl http://members.3322.org/dyndns/getip|
|ifconfig|Mac 查看本机内网IP 命令|ifconfig en0|
