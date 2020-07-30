---
title: iOS python自动化出包脚本
date: 2019-09-11 17:01:00
tags: 代码库
---

脚本代码如下：
```
#!/usr/bin/python
# -*- coding:UTF-8 -*-

import os
import sys
import time

# 发邮件所用
from email import encoders
from email.header import Header
from email.mime.text import MIMEText
from email.utils import parseaddr, formataddr
import smtplib

# 需要配置分割线 ===================================================================

# fir token
fir_api_token = '34d6f526c9fdcf9afe90753cdb9bb837' #firm的api token
download_address = "https://fir.im/xxxxxxxxx"      #firm 下载地址

# pgyer
pgyer_uKey         = "xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
pgyer_apiKey       = "xxxxxxxxxxxxxxxxxxxxxxxxxxxx"
pgyer_appQRCodeURL = "http://www.pgyer.com/xxxxxxxxx"   # 下载地址
pgyer_password     = "12345"
pgyer_updateDescription = "test版本"                     # 更新描述

# 项目配置
project_Name = 'Unity-iPhone'  #工程名
scheme = 'Unity-iPhone'        #scheme
isDistribution = False         #生成dev包或者dis包类型
isWorkspace = False            #工程类型 pod工程 -workspace 普通工程 -project

# 项目根目录
project_path = '/Users/yostar/Desktop/ProjectiOSTest'
#当前autoIpa.py 以及 plist 所在文件夹位置
#主执行文件的父级目录
autoPythonRoot = sys.path[0]

# 发邮件相关信息
from_addr = '250***2914@qq.com'
password = 'plgke***pzbjdice'
smtp_host = 'smtp.qq.com'
to_addr = ['250***2914@qq.com', '1728***24@qq.com']

# 需要配置分割线 ===================================================================

# 编译模式 Debug,Release
def configuration():
    if isDistribution:
        return 'Release'
    else:
        return 'Debug'

# 编译成功后.xcarchive所在目录
archive_dir = project_path + '/archive'
# 打包后ipa存储目录
targerIPA_dir = project_path + '/ipaDir'

#CA certificate
#发布包相关的plist
DistributionExportFileName = "Distribution_ExportOptions.plist"

#测试包相关的plist
DeveloperExportFileName = "Develop_ExportOptions.plist"

#时间字符串
time_Tag = '%s'%(time.strftime('%Y-%m-%d-%H-%M-%S', time.localtime(time.time())))

#xcodebuild export ipa包命令时需要用到
def export_OptionsPlist():
    if isDistribution:
        return autoPythonRoot + '/' + DistributionExportFileName
    else:
        return autoPythonRoot + '/' + DeveloperExportFileName

#打包名字
def archiveName():
    return project_Name + '_' + time_Tag + '.xcarchive'

#archive地址
def archivePath():
    return '%s/%s'%(archive_dir, archiveName())

#ipa包名
def ipaFileName():
    return '%s_%s'%(project_Name, time_Tag)

#ipa导出地址
def exportPath():
    if isDistribution:
        return '%s/%s/%s'%(targerIPA_dir, 'Distribution', ipaFileName())
    else:
        return '%s/%s/%s'%(targerIPA_dir, 'development', ipaFileName())

# 清理项目
def clean_project():
    os.system('rm -rf %s'%(archive_dir))
    print(project_path + '******' + project_Name + '******' + '******' + scheme + '******' + configuration())
    if isWorkspace:
        os.system('cd %s; xcodebuild clean -workspace %s.xcworkspace -scheme %s -configuration %s'%(project_path, project_Name, scheme, configuration()))
    else:
        os.system('cd %s; xcodebuild clean -project %s.xcodeproj -scheme %s -configuration %s'%(project_path, project_Name, scheme, configuration()))


#archive  打包
def archive_project():
    print('======archive_project start')
    print(archiveName())
    if isWorkspace:
        os.system('cd %s; xcodebuild archive -workspace %s.xcworkspace -scheme %s -archivePath %s'%(project_path, project_Name, scheme, archivePath()))
    else:
        os.system('cd %s; xcodebuild archive -project %s.xcodeproj -scheme %s -archivePath %s'%(project_path, project_Name, scheme, archivePath()))
   

# 打包ipa 并且保存在桌面
def export_ipa():
    print('export_ipa start')
    print(ipaFileName())
    print(export_OptionsPlist())
    os.system('cd %s; xcodebuild -exportArchive -archivePath %s/ -exportOptionsPlist %s -exportPath %s'%(project_path, archivePath(), export_OptionsPlist(), exportPath()))

##上传到fir
def upload_fir():
    p = exportPath() + '/' + scheme + '.ipa'
    if os.path.exists(p):
        print('watting===%s...上传到fir'%p)
        # 直接使用fir 有问题 这里使用了绝对地址 在终端通过 which fir 获得
        ret = os.system('fir publish %s -T %s'%(p, fir_api_token))
        print('watting...上传结束')
        return True
    else:
        print('没有找到IPA文件')
        return False

# 发邮件
def send_mail():
    msg = MIMEText('【%s】'%scheme + 'iOS 测试项目完成，请下载测试！如有问题，请联系iOS相关人员，我们会及时解决，谢谢!', 'plain', 'utf-8') #发邮件内容
    msg['From'] = Header('自动打包系统<%s>' % from_addr, 'utf-8') #发件人
    msg['To'] = Header('测试人员', 'utf-8') #收件人
    msg['Subject'] = Header('【%s】'%scheme + 'iOS客户端测试包构建完成, 构建时间:%s'%(time_Tag), 'utf-8').encode() #邮件主题
    
    try:
        server = smtplib.SMTP(smtp_host, 25)
        server.set_debuglevel(1)
        server.login(from_addr, password)
        server.sendmail(from_addr, to_addr, msg.as_string())
        print('邮件发送成功')
    except smtplib.SMTPException:
        print('Error:无法发送邮件')
    finally:
        server.quit() # 发送完毕后退出smtp

def main():
    # 执行
    # 清理目录
    clean_project()
    # 编译coocaPods项目文件并 执行编译目录
    archive_project()
    # 导出ipa
    export_ipa()

    if not isDistribution:
        # 上传fir
        success = upload_fir()
        # 发邮件
        if success:
            send_mail()

main()
```
* 该脚本是把`Distribution_ExportOptions.plist`、`Develop_ExportOptions.plist`和脚本放在同一目录里面的
* 该脚本是针对`xcode 8`及以上版本的，低版本会出包失败
* `plist`文件内容如下
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

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.09.17.01.jpeg)

* 其中`plist`文件中的`method`参数有如下几个方法：`{app-store, ad-hoc, enterprise, development}`
