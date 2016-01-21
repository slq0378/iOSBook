# 推送通知

## 本地推送

- 本地推送通知
![](/images/Snip20150814_45.png)

![](/images/Snip20150813_28.png)

- iOS8需要进行注册
```objc
    // iOS8进行本地通知的话，需要提前注册
    if([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0){
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge | UIUserNotificationTypeAlert | UIUserNotificationTypeSound categories:nil];
        // 注册
        [application registerUserNotificationSettings:settings];
    }
```
![](/images/Snip20150813_31.png)

![](/images/Snip20150814_43.png)

![](/images/Snip20150814_42.png)

- 第一次运行程序时会弹出提醒用户是否接收通知

![](/images/Snip20150813_27.png)

- iOS7会自己弹出
- alertView

![](/images/Snip20150813_29.png)
- 其他属性
![](/images/Snip20150813_32.png)

```objc
    // 1.创建本地通知
    UILocalNotification *localNote = [[UILocalNotification alloc] init];

    // 2.设置本地通知的内容
        // 2.1.设置通知发出的时间
        localNote.fireDate = [NSDate dateWithTimeIntervalSinceNow:3.0];
        // 2.2.设置通知的内容
        localNote.alertBody = @"吃饭了吗?";
        // 2.3.设置滑块的文字
        localNote.alertAction = @"快点";
        // 2.4.决定alertAction是否生效
        localNote.hasAction = NO;
        // 2.5.设置点击通知的启动图片
        localNote.alertLaunchImage = @"3213432dasf";
        // 2.6.设置alertTitle
        localNote.alertTitle = @"3333333333";
        // 2.7.设置有通知时的音效
        localNote.soundName = @"buyao.wav";
        // 2.8.设置应用程序图标右上角的数字
        localNote.applicationIconBadgeNumber = 999;

        // 2.9.设置额外信息
        localNote.userInfo = @{@"type" : @1};

    // 3.调用通知
    [[UIApplication sharedApplication] scheduleLocalNotification:localNote];
}
```

- 在appDelegate里

- 如果是在程序杀死情况下，需要在didFinishLaunchingWithOptions里做判断

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // iOS8进行本地通知的话，需要提前注册
    if([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0){
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge | UIUserNotificationTypeAlert | UIUserNotificationTypeSound categories:nil];
        // 注册
        [application registerUserNotificationSettings:settings];
    }
    // 设置消息数量位0
    [application setApplicationIconBadgeNumber:0];
    // 应用被杀死后，接收到通知会进入这个方法，然后根据通知做一些事情
    if (launchOptions[UIApplicationLaunchOptionsLocalNotificationKey]) {
        UILabel *lab = [[UILabel alloc] init];
        lab.numberOfLines = 0;
        lab.text = [NSString stringWithFormat:@"%@",launchOptions];
        lab.frame = self.window.bounds;
        [self.window.rootViewController.view addSubview:lab];
    }
    return YES;
}
```

![](/images/Snip20150814_44.png)

- 在应用程序不死的情况话，在前台或者由后台进入前台都会进入这个方法

```objc
/**
 *	在应用程序不死的情况话，在前台或者由后台进入前台都会进入这个方法
 */
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
{
    if (application.applicationState == UIApplicationStateInactive) {
        NSLog(@"由后台进入前台");
        NSLog(@"%@",notification.userInfo);
    }
    else if(application.applicationState == UIApplicationStateActive)
    {
         NSLog(@"在前台");
    }
    else if(application.applicationState == UIApplicationStateBackground)
    {
        NSLog(@"在后台"); // 在后台接收通知，不会执行
    }
}
```
#### 远程推送

- APNs
- deviceToken：获取必须有开发者账号，这个暂时没法测试了

![](/images/Snip20150814_46.png)

`格式:{"aps":{"alert":"This is some fancy message.","badge":1,"sound":"default"}}`

```objc
   if ([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0) { //iOS8
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound categories:nil];
        [application registerUserNotificationSettings:settings];
        // iOS8注册远程通知
        [application registerForRemoteNotifications];
    }
    // iOS7用这个方法注册
    [application registerForRemoteNotificationTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound];
```

![](/images/Snip20150813_35.png)

- 当得到苹果的APNs服务器返回的DeviceToken就会被调用

```objc
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    NSLog(@"deviceToken是：%@", deviceToken);
}
// 接收到远程通知，触发方法和本地通知一致
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    NSLog(@"%@", userInfo);
}
```

- 使用后台的远程消息推送

![](/images/Snip20150813_40.png)

```objc
1>	在Capabilities中打开远程推送通知
2> 实现代理方法
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler

远程消息数据格式：
{"aps" : {"content-available" : 1},"content-id" : 42}

执行completionHandler有两个目的
1> 系统会估量App消耗的电量，并根据传递的UIBackgroundFetchResult 参数记录新数据是否可用
2> 调用完成的处理代码时，应用的界面缩略图会自动更新

注意：接收到远程通知到执行完网络请求之间的时间不能超过30秒

if (userInfo) {
    int contentId = [userInfo[@"content-id"] intValue];

    ViewController *vc = (ViewController *)application.keyWindow.rootViewController;
    [vc loadDataWithContentID:contentId completion:^(NSArray *dataList) {
        vc.dataList = dataList;

        NSLog(@"刷新数据结束");

        completionHandler(UIBackgroundFetchResultNewData);
    }];
} else {
    completionHandler(UIBackgroundFetchResultNoData);
}
```

- deviceToken 转换字符串

```objc
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSLog(@"%@",deviceToken.description);
    NSString *receiveToken = [[deviceToken description] stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@"<>"]];
    receiveToken = [receiveToken stringByReplacingOccurrencesOfString:@" " withString:@""];
    
    NSString *localToken = [[NSUserDefaults standardUserDefaults] objectForKey:@"deviceToken"];
    if (![localToken isEqualToString:receiveToken]) {
        [[NSUserDefaults standardUserDefaults] setObject:receiveToken forKey:@"deviceToken"];
    }
    NSLog(@"%@",receiveToken);
}
```

- 在合适的时间，比如登陆成功后上传至自己的服务器

- 第三方apns
	- JPush
	- <https://www.jpush.cn>
