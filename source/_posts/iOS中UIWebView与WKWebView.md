---
title: iOS 中UIWebView与WKWebView
date: 2019-02-14 15:58:00
tags: 代码库
---

# 1. UIWebView
UIWebView 适用于iOS8.0以下的系统版本
iOS原生没有提供js直接调用OC的方式，只能通过UIWebView的UIWebViewDelegate协议方法来做拦截，并在这个方法中，根据url来调用OC方法；
```
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
```
* UIWebViewDelegate代理协议使用：
```
// 是否允许加载网页，也可获取js要打开的url，通过截取此url可与js交互
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    NSString *urlString = [[request URL] absoluteString];
    urlString = [urlString stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

    NSArray *urlComps = [urlString componentsSeparatedByString:@"://"];
    NSLog(@"urlString=%@---urlComps=%@",urlString,urlComps);
    return YES;
}
// 开始加载网页
- (void)webViewDidStartLoad:(UIWebView *)webView {
    NSURLRequest *request = webView.request;
    NSLog(@"webViewDidStartLoad-url=%@--%@",[request URL],[request HTTPBody]);
}
// 网页加载完成
- (void)webViewDidFinishLoad:(UIWebView *)webView {
    NSURLRequest *request = webView.request;
    NSURL *url = [request URL];
    if ([url.path isEqualToString:@"/normal.html"]) {
        NSLog(@"isEqualToString");
    }
    NSLog(@"webViewDidFinishLoad-url=%@--%@",[request URL],[request HTTPBody]);
    NSLog(@"%@",[self.webView stringByEvaluatingJavaScriptFromString:@"document.title"]);
}
// 网页加载错误
- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error {
    NSURLRequest *request = webView.request;
    NSLog(@"didFailLoadWithError-url=%@--%@",[request URL],[request HTTPBody]);
}
```

# 2. WKWebView

* 引入WebKit库
`#import <WebKit/WebKit.h>`
* 在文档中可以看出，WKWebView的初始化方法有两种：
```
- (instancetype)initWithFrame:(CGRect)frame
- (instancetype)initWithFrame:(CGRect)frame configuration:(WKWebViewConfiguration *)configuration
```
我们大多使用第一种方式，这也是文档中默认的一种方式。
* WKWebView 代理
协议方法如下：
```
//页面开始加载时调用
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(null_unspecified WKNavigation *)navigation{
    NSLog(@"页面开始加载时调用。   2");
}
//内容返回时调用，得到请求内容时调用(内容开始加载) -> view的过渡动画可在此方法中加载
- (void)webView:(WKWebView *)webView didCommitNavigation:( WKNavigation *)navigation{
    NSLog(@"内容返回时调用，得到请求内容时调用。 4");
}
//页面加载完成时调用
- (void)webView:(WKWebView *)webView didFinishNavigation:( WKNavigation *)navigation{
    NSLog(@"页面加载完成时调用。 5");
}
//请求失败时调用
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation withError:(NSError *)error{
    NSLog(@"error1:%@",error);
}
-(void)webView:(WKWebView *)webView didFailNavigation:(WKNavigation *)navigation withError:(NSError *)error{
    NSLog(@"error2:%@",error);
}
//在请求发送之前，决定是否跳转 -> 该方法如果不实现，系统默认跳转。如果实现该方法，则需要设置允许跳转，不设置则报错。
//该方法执行在加载界面之前
//Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'Completion handler passed to -[ViewController webView:decidePolicyForNavigationAction:decisionHandler:] was not called'
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler{
    //允许跳转
    decisionHandler(WKNavigationActionPolicyAllow);

    //不允许跳转
    //    decisionHandler(WKNavigationActionPolicyCancel);
    NSLog(@"在请求发送之前，决定是否跳转。  1");
}
//在收到响应后，决定是否跳转（同上）
//该方法执行在内容返回之前
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler{
    //允许跳转
    decisionHandler(WKNavigationResponsePolicyAllow);
    //不允许跳转
    //    decisionHandler(WKNavigationResponsePolicyCancel);
    NSLog(@"在收到响应后，决定是否跳转。 3");

}
//接收到服务器跳转请求之后调用
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(null_unspecified WKNavigation *)navigation{
    NSLog(@"接收到服务器跳转请求之后调用");
}
-(void)webViewWebContentProcessDidTerminate:(WKWebView *)webView{
    NSLog(@"webViewWebContentProcessDidTerminate");
}
```
# 3. OC调用JS传递参数
* UIWebView方法
```
NSString *str = [self.webview stringByEvaluatingJavaScriptFromString:[NSString stringWithFormat:@"postStr('%@','%@');",str1,str2]];
```
* WKWebView方法
```
[self.wkWebView evaluateJavaScript:[NSString stringWithFormat:@"postStr('%@')", @"true"] completionHandler:^(id _Nullable result, NSError * _Nullable error) {
    NSLog(@"==%@----%@", result, error);
}];
```
在需要接受参数值的js界面实现如下方法:
```
function postStr(str1, str2){
    alert(str1, str2);    //接收到的值”;
    //...code
}
```
# 4. JS调用OC传递参数
js是不能执行oc代码的，但是可以变相的执行，js可以将要执行的操作封装到网络请求里面，然后oc拦截这个请求，获取url里面的字符串解析即可。（拦截URL）
比如darkangel://。方法是在html或者js中，点击某个按钮触发事件时，跳转到自定义URL Scheme构成的链接，而OC中捕获该链接，从中解析必要的参数，实现JS到OC的一次交互。比如页面中一个a标签，链接如下：
`<a href="darkangel: smslogin?username="12323123&code=892845"">短信验证登录</a href="darkangel:>`
在该方法中，捕获该链接，并且返回NO（阻止本次跳转），从而执行对应的OC方法。
* UIWebView方法
```
/*
* 方法的返回值是BOOL值。
* 返回YES：表示让浏览器执行默认操作，比如某个a链接跳转
* 返回NO：表示不执行浏览器的默认操作，这里因为通过url协议来判断js执行native的操作，肯定不是浏览器默认操作，故返回NO
*/
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
    //标准的URL包含scheme、host、port、path、query、fragment等
    NSURL *URL = request.URL;    
    if ([URL.scheme isEqualToString:@"darkangel"]) {
        if ([URL.host isEqualToString:@"smsLogin"]) {
            NSLog(@"短信验证码登录，参数为 %@", URL.query);
            return NO;
        }
    }
    return YES;
}
```
* WKWebView方法
```
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    //可以通过navigationAction.navigationType获取跳转类型，如新链接、后退等
    NSURL *URL = navigationAction.request.URL;
    //判断URL是否符合自定义的URL Scheme
    if ([URL.scheme isEqualToString:@"darkangel"]) {
        //根据不同的业务，来执行对应的操作，且获取参数
        if ([URL.host isEqualToString:@"smsLogin"]) {
            NSString *param = URL.query;
            NSLog(@"短信验证码登录, 参数为%@", param);
            //不允许跳转
            decisionHandler(WKNavigationActionPolicyCancel);
            return;
        }
    }
    //允许跳转
    decisionHandler(WKNavigationActionPolicyAllow);
    NSLog(@"%@", NSStringFromSelector(_cmd));
}
```
