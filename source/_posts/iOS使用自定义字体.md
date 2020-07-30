---
title: iOS 使用自定义字体
date: 2019-07-24 15:32:00
tags: 代码库
---

**1. 动态下载系统提供的中文字体**
* 为了实现更好的字体效果，在应用中加入字体包问题：
> 1. 字体文件比较大，会造成应用体积剧增
> 2. 中文字体通常都是有版权的
* 动态下载中文字体的API可以动态的向iOS系统中添加字体文件，这些字体文件都是下载到系统的目录中，所以不会造成应用体积增加，字体文件下载后还可以在所有应用间共享。并且字体文件是iOS系统提供的，也免去了字体使用版权的问题
* 下载的时候需要使用的名字是 PostScript 名称，所以你要动态下载相应的字体的话，还需要使用 Mac 内自带的应用 “字体册 “来获得相应字体的 PostScript 名称。如下显示了从” 字体册 “中获取字体的 PostScript 名称的截图
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.07.24.01.png)

* 苹果提供的动态下载代码的 [Demo 工程](http://developer.apple.com/library/ios/#samplecode/DownloadFont/Listings/DownloadFont_ViewController_m.html) 链接在这里。将此 Demo 工程下载下来，即可学习相应 API 的使用
```
// 1. 先判断该字体是否已经被下载下来了
- (BOOL)isFontDownloaded:(NSString *)fontName {
    UIFont* aFont = [UIFont fontWithName:fontName size:12.0];
    if (aFont && ([aFont.fontName compare:fontName] == NSOrderedSame 
               || [aFont.familyName compare:fontName] == NSOrderedSame)) {
        return YES;
    } else {
        return NO;
    }
}

// 2. 如果该字体下载过了，则可以直接使用。否则需要先准备下载字体 API 需要的一些参数
// 用字体的 PostScript 名字创建一个 Dictionary
NSMutableDictionary *attrs = [NSMutableDictionary dictionaryWithObjectsAndKeys:fontName, kCTFontNameAttribute, nil];
// 创建一个字体描述对象 CTFontDescriptorRef
CTFontDescriptorRef desc = CTFontDescriptorCreateWithAttributes((__bridge CFDictionaryRef)attrs);
// 将字体描述对象放到一个 NSMutableArray 中
NSMutableArray *descs = [NSMutableArray arrayWithCapacity:0];
[descs addObject:(__bridge id)desc];
CFRelease(desc);

// 3. 准备好上面的descs变量后，则可以进行字体的下载了
__block BOOL errorDuringDownload = NO;
CTFontDescriptorMatchFontDescriptorsWithProgressHandler((CFArrayRef)descs, NULL, ^bool(CTFontDescriptorMatchingState state, CFDictionaryRef  _Nonnull progressParameter){
    
    double progressValue = [[(__bridge NSDictionary *)progressParameter objectForKey:(id)kCTFontDescriptorMatchingPercentage] doubleValue];
    
    if (state == kCTFontDescriptorMatchingDidBegin) {
        NSLog(@" 字体已经匹配 ");
    } else if (state == kCTFontDescriptorMatchingDidFinish) {    
        if (!errorDuringDownload) {
            NSLog(@" 字体 %@ 下载完成 ", fontName);
        }
    } else if (state == kCTFontDescriptorMatchingWillBeginDownloading) {
        NSLog(@" 字体开始下载 ");
    } else if (state == kCTFontDescriptorMatchingDidFinishDownloading) {
        NSLog(@" 字体下载完成 ");
        dispatch_async( dispatch_get_main_queue(), ^ {
            // 可以在这里修改 UI 控件的字体
            // self.label.font = [UIFont fontWithName:fontName size:12];
        });
    } else if (state == kCTFontDescriptorMatchingDownloading) {
        NSLog(@" 下载进度 %.0f%%", progressValue);
    } else if (state == kCTFontDescriptorMatchingDidFailWithError) {
        NSError *error = [(__bridge NSDictionary *)progressParameter objectForKey:(id)kCTFontDescriptorMatchingError];
        if (error != nil) {
            _errorMessage = [error description];
        } else {
            _errorMessage = @"ERROR MESSAGE IS NOT AVAILABLE!";
        }
        // 设置标志
        errorDuringDownload = YES;
        NSLog(@" 下载错误: %@", _errorMessage);
    }
    return YES;
});
```
**2. 导入TTF字体文件使用自定义字体**
* 下载字体导入工程（**注意上文说的字体版权问题**）
* 在 info.plist文件中告诉系统你想导入的字体文件
![方正兰亭纤黑](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.07.24.02.png)

* 设置字体到相应控件上（使用的字体名字是 PostScript 名称，上文已说过怎样获取 PostScript 名称）
```
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(10, 100, 300, 400)];
label.text = @"汉体书写信息技术标准相容档案下载使用界面简单";
label.numberOfLines = 0;
UIFont *font = [UIFont fontWithName:@"FZLTXHK--GBK1-0" size:40];
[self.view addSubview:label];
```
