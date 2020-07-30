---
title: mac 搭建基于Hexo-Github的Blog
date: 2019-08-14 15:56:00
tags: 代码库
---

* GitHubPages + Hexo：免费，使用简单，使用者众多

**博客搭建**
1. **创建 GitHub 仓库**
![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.01.jpeg)

* 注意 Respository name 中一定要输入：`你的用户名.github.io`其他地方不用修改，然后直接点 ”Create repository“ 按钮完成创建即可
2. **安装博客需要的框架**
* 安装 Homebrew
`$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
* 查询Homebrew是否安装成功的命令
`$ brew -v`
* 安装 git
`$ brew install git`
* 查询git是否安装成功的命令
`$ git –version`
* 安装 node.js
`$ brew install node`
* 查询node是否安装成功的命令
`$ node -v`
* 安装 hexo
`$ npm install -g hexo`
* 查询hexo是否安装成功的命令
`$ hexo -v`
3. **安装博客相关插件**
* 自动部署到Github上的插件
`$ npm install hexo-deployer-git --save`
* 安装atom生成插件，便于感兴趣的小伙伴们订阅(RSS订阅)
`$ npm install hexo-generator-feed --save`
然后在本地Blog根目录下的_config.yml文件中，添加以下配置
```
# Extensions
## Plugins: http://hexo.io/plugins/
#RSS订阅
plugin: -hexo-generator-feed
#Feed Atom
feed:
     type: atom
     path: atom.xml
     limit: 20
```
在主题目录下的_config.yml目录下，添加如下配置
`rss: /atom.xml`
* 安装博客首页生成插件
`$ npm install hexo-generator-index --save`
* 安装tag生成插件
`$ npm install hexo-generator-tag --save`
**到此为止，安装完毕**
4. **创建博客、调试、发布**
* 创建本地一个目录，用来创建博客
`$ hexo init Blog`
执行成功后，会创建出一个名为Blog的文件夹
* 撰写博客内容
cd到Blog文件夹下，执行命令
`$ hexo new firstBlog`
在Blog/source/_posts中就会新建一个firstBlog.md的文件，然后你就可以编辑你的博客内容了，Visual Studio Code编辑器支持预览，还可以和印象笔记同步
* 本地调试
`$ sudo hexo server 或 $ sudo hexo s`
然后可以访问`http://localhost:4000`来查看结果
* 安装发布插件
在博客文件夹运行下面命令
`$ npm install hexo-deployer-git --save`
然后在_config.yml文件修改发布的配置，最后一行改为（注意替换yourusername）
```
# Site
title: 博客的名字
subtitle: 博客副标题
description: 博客描述
author: 博客作者
language: zh-Hans
theme: next //安装的主题名称
deploy:
  type: git  //使用Git 发布
  repo: https://github.com/yourusername/yourusername.github.com.git  //自己的Github仓库地址
  branch: master
```
* 运行生成发布
```
$ sudo hexo g
$ sudo hexo d
```
* 如果改动了站点的源码，需要在发布之前
`$ sudo hexo clean`
* 如果成功了就可以通过`yourusername.github.com`或者`yourusername.github.io`来访问你的博客了
5. **配置博客分类内容**
* 新增资源分类页面
```
hexo new page about
INFO  Created: ~/blog/source/about/index.md
// 打开修改为如下
---
title: about
date: 2016-09-17 13:21:20
tags: 代码库        // 标签
comment: false   // 添加这行关闭评论
---
here is something about me
```
* 新添加的菜单需要翻译对应的中文
打开`\themes\next\languages`文件夹，创建主站配置语言的对应文件，如：zh-Hans.yml
6. **自定义博客**
* 更换主题
如果你对默认的主题不满意，可以通过克隆的方式，把别人的主题克隆到项目"/themes"路径下，喵神的主题在[这里](https://link.jianshu.com?t=https://github.com/monniya/hexo-theme-new-vno)，Hexo也有[更多主题](https://link.jianshu.com?t=https://hexo.io/themes/)，比如使用这一套
`git clone https://github.com/iissnan/hexo-theme-next.git themes/next`
然后在blog/themes文件夹下，会看到一个next文件夹，在next中有一个_config.yml文件，它就是主题配置文件，这里略过[设置详情](https://link.jianshu.com?t=http://theme-next.iissnan.com/getting-started.html#theme-settings)；要设置博客主题的话，还是要回到根目录下(Blog文件夹下)的_config.yml文件中，将theme属性由landscape修改为next，还有注意配置的键值之间一定要有**空格**。[更多设置](https://link.jianshu.com?t=https://hexo.io/zh-cn/docs/configuration.html)
修改并保存之后，我们再执行命令
```
$ hexo g
$ hexo d
```
执行完毕以后，就可以在本地或者git上看到博客的新主题了，更多的主题可以[参考知乎](https://link.jianshu.com/?t=https://www.zhihu.com/question/24422335)
* 设置网页浏览次数
打开themes/你的主题/layout/_partial/footer.ejs添加即可
```
# 脚本
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>

# 标签
# pv的方式，单个用户连续点击n篇文章，记录n次访问量
<span id="busuanzi_container_site_pv">
    本站总访问量<span id="busuanzi_value_site_pv"></span>次
</span>
# uv的方式，单个用户连续点击n篇文章，只记录1次访客数
<span id="busuanzi_container_site_uv">
     总访客数<span id="busuanzi_value_site_uv"></span>人
</span>
```

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.02.jpg)

* 评论功能
使用韩国来必力评论，需要注册账号[详见官网](https://www.livere.com/)
在主题配置文件下添加
`page_comment: true`
在`\themes\random\layout\_partial`文件夹中新建`livere.ejs`文件并添加如下代码
```
<section class="livere" id="comments">
    <!-- 来必力City版安装代码 -->
    <div id="lv-container" data-id="city" data-uid="MTAyM***注册成功官方返回的uid****xNzkwMw==">
      <script type="text/javascript">
       (function(d, s) {
           var j, e = d.getElementsByTagName(s)[0];

           if (typeof LivereTower === 'function') { return; }

           j = d.createElement(s);
           j.src = 'https://cdn-city.livere.com/js/embed.dist.js';
           j.async = true;

           e.parentNode.insertBefore(j, e);
       })(document, 'script');
      </script>
    <noscript> 为正常使用来必力评论功能请激活JavaScript</noscript>
    </div>
    <!-- City版安装代码已完成 -->
</section>
```

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.03.jpeg)

在`\themes\random\layout\post.ejs`文件中添加或修改如下代码
```
<% if (page.comment || theme.page_comment) { %>
  <%- partial('_partial/livere',{}) %>
<% } %>
```
* 顶部加载条
在`\themes\random\layout\_partial head.swig或head.ejs`文件中添加如下代码
```
<script src="//cdn.bootcss.com/pace/1.0.2/pace.min.js"></script>
<link href="//cdn.bootcss.com/pace/1.0.2/themes/pink/pace-theme-flash.css" rel="stylesheet">
  <style>
      .pace .pace-progress {
          background: #1E92FB; /*进度条颜色*/
          height: 3px;
      }
      .pace .pace-progress-inner {
           box-shadow: 0 0 10px #1E92FB, 0 0 5px     #1E92FB; /*阴影颜色*/
      }
      .pace .pace-activity {
          border-top-color: #1E92FB;    /*上边框颜色*/
          border-left-color: #1E92FB;    /*左边框颜色*/
      }
  </style>
```

![添加在head里面](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.04.png)

* 安全运行天数
在`\themes\random\layout\_partial footer.swig或footer.ejs`页脚部分添加如下代码
```
# 标签
<span id="timeDate">载入天数...</span><span id="times">载入时分秒...</span>
# 脚本
<script>
    var now = new Date(); 
    function createtime() { 
        var grt= new Date("07/08/2018 12:00:00");//此处修改你的建站时间或者网站上线时间 
        now.setTime(now.getTime()+250); 
        days = (now - grt ) / 1000 / 60 / 60 / 24; dnum = Math.floor(days); 
        hours = (now - grt ) / 1000 / 60 / 60 - (24 * dnum); hnum = Math.floor(hours); 
        if(String(hnum).length ==1 ){hnum = "0" + hnum;} minutes = (now - grt ) / 1000 /60 - (24 * 60 * dnum) - (60 * hnum); 
        mnum = Math.floor(minutes); if(String(mnum).length ==1 ){mnum = "0" + mnum;} 
        seconds = (now - grt ) / 1000 - (24 * 60 * 60 * dnum) - (60 * 60 * hnum) - (60 * mnum); 
        snum = Math.round(seconds); if(String(snum).length ==1 ){snum = "0" + snum;} 
        document.getElementById("timeDate").innerHTML = "本站已安全运行 "+dnum+" 天 "; 
        document.getElementById("times").innerHTML = hnum + " 小时 " + mnum + " 分 " + snum + " 秒"; 
    } 
    setInterval("createtime()",250);
</script>
```

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.05.png)

* 在每篇文章末尾统一添加“本文结束”标记
接着打开`\themes\vno\layout\post.ejs`文件，在post-body之后， post-footer之前添加如下代码
```
<div>
    <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束<i class="fa fa-paw"></i>感谢您的阅读-------------</div>
</div>  
```
* 添加打赏功能，使用iframe嵌入打赏
把下面代码添加到你想要的位置，一般是在`<article>`标签内，如`\themes\vno\layout\post.ejs`的<article>标签内

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.06.png)

`<iframe src="https://gsl201600.github.io/RewardCode" style="overflow-x:hidden;overflow-y:hidden; border:0xp none #fff; min-height:240px; width:100%;"  frameborder="0" scrolling="no" allowtransparency="true"></iframe>`
把该项目添加到你的网站，修改相关参数为你的打赏码，即可实现[项目地址](https://github.com/Gsl201600/RewardCode)
* 在Hexo博客上添加可爱的Live 2D模型
安装npm包
`$ npm install --save hexo-helper-live2d`
然后在Blog的配置文件`_config.yml`中添加如下配置，详细配置可以参考[文档](https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md)
```
live2d:
  enable: true
  scriptFrom: local
  pluginRootPath: live2dw/
  pluginJsPath: lib/
  pluginModelPath: assets/
  tagMode: false
  debug: false
  model:
    use: live2d-widget-model-shizuku
  display:
    position: right
    width: 150
    height: 300
  mobile:
    show: true
```
然后下载模型，模型名称可以到[这里](https://github.com/xiazeyu/live2d-widget-models)参考，一些模型的预览可以在[这里](https://huaji8.top/post/live2d-plugin-2.0/)
`$ npm install live2d-widget-model-shizuku`
下载完之后，在Blog根目录中新建文件夹live2d_models，然后在node_modules文件夹中找到刚刚下载的live2d模型，将其复制到live2d_models中，然后编辑配置文件中的model.use项，将其修改为live2d_models文件夹中的模型文件夹名称
一切就绪之后，用hexo server命令启动服务器，稍等一下就可以看到右下角出现了一个可爱的萌萌哒的妹纸！不过因为所有东西都在Github上托管的原因，可能Live2D不能马上加载出来
* Hexo操作指令一览
```
$ hexo clean      #清理缓存
$ hexo generate   #生成静态文件
$ hexo server     #启动本地服务器
$ hexo deploy    #部署
或者
$ hexo clean      #清理缓存
$ hexo g          #生成静态文件
$ hexo s          #启动本地服务器
$ hexo d         #部署
```
* homebrew操作指令一览
```
// 安装 homebrew
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
// 卸载 homebrew
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
// 查看版本
$ brew -v
// 下载软件：brew install 软件名
如：$ brew install htop, 安装htop
// 如需要图形安装的软件 要加cask
如：$ brew cask install google-chrome
// 卸载软件:brew uninstall 软件名
如: $ brew cask uninstall google-chrome
// 软件搜索:brew search 软件名
如: $ brew search google
// 列出已安装的包
$ brew list
// 查看软件相关信息:brew info 软件名
如：$ brew info google-chrome
// 删除 Homebrew下载的包
$ brew cleanup
// 更新 Homebrew
$ brew update
```
* **附：给Hexo搭建的博客增加百度和谷歌的搜索引擎验证**
1. 验证站点
搜索引擎验证的方法有好几种，下面选择 `HTML标签验证` 验证方法，其他的方法有兴趣可以自己去试一下
* 首先打开 [百度搜索引擎验证](https://link.jianshu.com?t=http://www.baidu.com/search/url_submit.htm) ，点击 `添加网站` ，输入自己的 `博客` 地址
* 输完后选择 `HTML标签验证` ，然后将下方的 `meta` 代码复制下来，网页先不要关

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.07.png)

* 重新开一个页面，打开[谷歌搜索引擎验证](https://link.jianshu.com/?t=https://www.google.com/webmasters/tools/home?hl=zh-CN)，点击`添加属性`，一样输入自己的`博客`地址
* 输完后选择`备用方法`下的`HTML 标记`，然后将下方的`meta`代码复制下来，网页也不要关

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.08.png)

* 打开本地博客主题下的layout/_partial 文件夹，有一个名为head的文件，使用HTML编辑器打开，将刚才复制的两句meta代码粘贴进去
* 保存文件后，输入以下命令将博客重新部署到GitHub服务器
`hexo clean && hexo g && hexo d`
* 然后分别点击刚才百度、谷歌验证页面的验证按钮进行站点验证
2. 生成站点地图
* 打开终端cd到本地博客目录下，输入以下命令安装sitmap插件
```
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```
* 打开本地博客目录下的_config.yml文件，修改url参数为你博客的首页地址，这样是为了保证能正确生成sitemap.xml文件中的地址
```
url: http://jonzzs.cn # 修改成你博客的首页地址
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```
* 添加以下配置
```
# 自动生成sitemap
sitemap: 
  path: sitemap.xml
baidusitemap: 
  path: baidusitemap.xml
```
* 输入以下命令重新部署博客
`hexo clean && hexo g && hexo d`
3. 将站点地图提交谷歌
* 打开[谷歌站点控制台](https://link.jianshu.com/?t=https://www.google.com/webmasters/tools/home?hl=zh-CN)进入站点控制台，先点击`测试`站点地图，测试通过后再点击`提交`站点地图

![](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.08.14.09.png)

* 百度主动推送
首先，在Hexo根目录下，安装本插件：`$ npm install hexo-baidu-url-submit --save`
然后，同样在根目录下，把以下内容配置到_config.yml文件中:
```
baidu_url_submit:
  count: 3 ## 提交最新的3个链接
  host: www.hui-wang.info ## 在百度站长平台中注册的域名
  token: your_token ## 请注意这是您的秘钥， 所以请不要把博客源代码发布在公众仓库里!
  path: baidu_urls.txt ## 文本文档的地址， 新链接会保存在此文本文档里
```
其次，记得查看_config.ym文件中url的值， 必须包含是百度站长平台注册的域名，比如:
```
# URL
url: http://www.hui-wang.info
root: /
permalink: :year/:month/:day/:title/
```
最后，加入新的deployer:
```
deploy:
- type: git ## 这是原来的deployer
  repo: https://github.com/yourusername/yourusername.github.com.git  //自己的Github仓库地址
  branch: master
- type: baidu_url_submitter ## 这是新加的
```
执行hexo deploy的时候，新的连接就会被推送了
* 百度自动推送
安装自动推送JS代码的网页，在页面被访问时，页面URL将立即被推送给百度，修改主题目录下的layout/post.swig文件，末尾添加自动推送代码，代码如下
```
<script>
(function(){
    var bp = document.createElement('script');
    var curProtocol = window.location.protocol.split(':')[0];
    if (curProtocol === 'https') {
        bp.src = 'https://zz.bdstatic.com/linksubmit/push.js';        
    }
    else {
        bp.src = 'http://push.zhanzhang.baidu.com/push.js';
    }
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(bp, s);
})();
</script>
```
4. 用`site:域名`测试是否成功
