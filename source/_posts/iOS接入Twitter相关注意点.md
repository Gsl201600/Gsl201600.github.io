---
title: iOS 接入Twitter 相关注意点
date: 2019-04-23 16:56:00
tags: 代码库
---

**1. 接入前配置**
* Download and unzip [Twitter Kit](https://ton.twimg.com/syndication/twitterkit/ios/3.3.0/Twitter-Kit-iOS.zip)
* Add `TwitterKit` to "Embedded Binaries" in your Xcode project settings(测试发现不添加也可以)
* Add `TwitterKit` and `TwitterCore` to "Linked Frameworks and Libraries" in your Xcode project settings
* Add `SafariServices.framework` to use SFSafariViewController
* In your app's Info.plist, add URL Schemes by adding code below after
```
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>twitterkit-<consumerKey></string>
    </array>
  </dict>
</array>
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>twitter</string>
    <string>twitterauth</string>
</array>
```
* Make sure to import the framework header: `#import <TwitterKit/TWTRKit.h>`
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [[Twitter sharedInstance] startWithConsumerKey:@"hTpkPVU4pThkM0" consumerSecret:@"ovEqziMzLpUOF163Qg2mj"];
}
```
* Implement the application:openURL:options method in your Application Delegate, and pass along the redirect URL to Twitter Kit
```
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options {
 return [[Twitter sharedInstance] application:app openURL:url options:options];
}
```

**2. Twitter后台配置 [https://apps.twitter.com/app](https://apps.twitter.com/app)**

![Twitter apps dashboard](https://raw.githubusercontent.com/Gsl201600/PicGoImg/master/img/2019.04.23.01.png)

**From June 12th 2018 callback locking will [no longer be optional](https://developer.twitter.com/en/docs/basics/callback_url.html). The correct callback format for iOS apps is:`twitterkit-MY_CONSUMER_KEY://`**

**3. 接入相关功能**
* **Log In Button**
```
TWTRLogInButton *logInButton = [TWTRLogInButton buttonWithLogInCompletion:^(TWTRSession *session, NSError *error) {
  if (session) {
      NSLog(@"signed in as %@", [session userName]);
  } else {
      NSLog(@"error: %@", [error localizedDescription]);
  }
}];
logInButton.center = self.view.center;
[self.view addSubview:logInButton];
```
* **Log In Method**
```
[[Twitter sharedInstance] logInWithCompletion:^(TWTRSession *session, NSError *error) {
  if (session) {
      NSLog(@"signed in as %@", [session userName]);
  } else {
      NSLog(@"error: %@", [error localizedDescription]);
  }
}];
```
* **Request User Email Address**
```
TWTRAPIClient *client = [TWTRAPIClient clientWithCurrentUser];
[client requestEmailForCurrentUser:^(NSString *email, NSError *error) {
  if (email) {
      NSLog(@"signed in as %@", email);
  } else {
      NSLog(@"error: %@", [error localizedDescription]);
  }
}];
```
