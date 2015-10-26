# 真机调试-分享-推送通知


## 真机调试

![](../images/屏幕快照 2015-08-13 09.05.55.png)
- 证书
	- 1. iOS dev
	- 2. 创建CSR，证书请求文件
	- 3. 在钥匙串访问中，申请，从授权机构申请证书－》保存到磁盘
	- 4. 将CSR文件上传到苹果服务器
	- 5. 苹果服务器会自动生成，签名后的CER文件
- App IDs
- 设备
- 生成描述文件

## 发布程序

![](../images/Snip20150813_10.png)

- LaunchScreen.xib
	- 自动进行屏幕适配
- 过程
	- 1、准备好程序
	- 2、配置证书
		- 1、生成本机证书
		- 2、指定App IDs
		- 3、生成描述文件
	- 3、
- 内购
	- 框架 `StoreKit`
	- 去服务器请求产品ID
	- 去苹果服务器请求可买商品 `SKProductsRequest`
		- 设代理获取返回的参数 `SKProductsRequestDelegate`
		- `sortedArrayWithOptions`
		- 将返回的可买商品显示出来,默认返回数据时无序的，按照价格排序

```objc
- (void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response
{
	// 展示商品
	self.products = [response.products sortedArrayWithOptions:NSSortConcurrent usingComparator:^NSComparisonResult(SKProduct *obj1, SKProduct *obj2) {
	    return [obj1.price compare:obj2.price];
	}];
}
```
	- 添加观察者，监视用户购买事件
	- 购买商品

```objc
// 购买商品
- (void)buyProduct:(SKProduct *)product
{
    // 1.创建票据
    SKPayment *payment = [SKPayment paymentWithProduct:product];

    // 2.将票据加入到交易队列中
    [[SKPaymentQueue defaultQueue] addPayment:payment];
}
```
		- `SKPayment`创建票据，将票据添加到交易队列 `SKPaymentQueue`
	- 监听交易队列的改变 `SKPaymentTransactionObserver`

```objc
/*
 队列中的交易发生改变时,就会调用该方法
 */
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions
{
    /*
     SKPaymentTransactionStatePurchasing,    正在购买
     SKPaymentTransactionStatePurchased,     已经购买(购买成功)
     SKPaymentTransactionStateFailed,        购买失败
     SKPaymentTransactionStateRestored,      恢复购买
     SKPaymentTransactionStateDeferred       未决定
     */
    for (SKPaymentTransaction *transation in transactions) {
        switch (transation.transactionState) {
            case SKPaymentTransactionStatePurchasing:
                NSLog(@"用户正在购买");
                break;

            case SKPaymentTransactionStatePurchased:
                NSLog(@"购买成功,将对应的商品给用户");

                // 将交易从交易队列中移除
                [queue finishTransaction:transation];
                break;

            case SKPaymentTransactionStateFailed:
                NSLog(@"购买失败,告诉用户没有付钱成功");

                // 将交易从交易队列中移除
                [queue finishTransaction:transation];
                break;

            case SKPaymentTransactionStateRestored:
                NSLog(@"恢复商品,将对应的商品给用户");
                // transation.payment.productIdentifier
                // 将交易从交易队列中移除
                [queue finishTransaction:transation];
                break;

            case SKPaymentTransactionStateDeferred:
                NSLog(@"未决定");
                break;
            default:
                break;
        }
    }
}
```
	- 如果已经购买过

```objc
// 恢复购买
- (IBAction)restore:(id)sender {
    [[SKPaymentQueue defaultQueue] restoreCompletedTransactions];
}
```
- 广告 `ADBannerView`
	- 使用广告控件的话，必须手动添加`iAd.framework`
	- 使用代理监视广告加载状态 `ADBannerViewDelegate`
- admo

- 内购总结
	- 1.配置⼀一个明确的APPID
	- 2.配置内购相关的内容
		- 配置内购项⺫⽬目
		- 配置测试账号
		- 配置银⾏行信息
	- 3.内购的流程
	- 4.内购代码的实现
```objc
		获取想卖的商品的ProductID(服务器)—>NSSet 
		请求可买的商品(SKProductRequest) 
		代理⽅方法:SKProductResponse—>products—>SKProduct 
		SKPayment paymentWithProduct:product 
		[SKPaymentQueue defaultQueue] addPayment:payment 
		[添加观察者]
		SKPaymentTransaction—>transactionState
		[SKPaymentQueue defaultQueue] retoreTransactions
```
- 测试打包

## 应用间跳转

![](../images/应用间跳转URL的添加方式.gif)

- 先跳转到其他应用，操作完成后再跳转回来。
- 通过URL打开其他应用
	- 直接通过协议头打开
	- 在info.list里添加`URL Type`
- 监听应用的代理方法：获取是哪个URL打开的应用
	- `appDelegate openUrl：`

```objc
    // 1、一定要写出来 "yueyue://"
    NSURL *yueyue = [NSURL URLWithString:@"yueyue://"];
    // 2、先判断是否可以打开
    if ([[UIApplication sharedApplication] canOpenURL:yueyue]) {
        // 3、打开URL
        [[UIApplication sharedApplication] openURL:yueyue];
    }
```

![](../images/Snip20150813_12.png)

- 跳转回去
	- 在跳转之前将自己URL包装起来，传递到下一个应用，然后由下一个应用获取并跳转回来

- AppDelegate

```objc
/**
 *  旧方法
 *	其他应用调用这个应用都会进入这个方法，并且URL是当前应用的URL
 */
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
{
    NSString *urlStr = url.absoluteString;
    NSLog(@"%@",urlStr);
    return YES;
}

/**
 *	新方法，重写这个，旧方法就不会调用了
 *  sourceApplication 是当开当前应用程序的源程序的BundleId
 */
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    NSString *urlStr = url.absoluteString;
    NSLog(@"%@--%@",urlStr,sourceApplication);
    return YES;
}
```

- 进入应用的子控制器
	-改变协议的路径,通过协议头后面的文字区别来在应用中判断要跳转的子控制器
`[self openAppWithString:@"yueyue://public"];
 [self openAppWithString:@"yueyue://private"];`

	- 而且要在openURL对要判断的字符串进行保存，传递到其他控制器。传递方式通知或者代理都不行，只能使用属性传值

```objc
/**
 *	新方法，重写这个，旧方法就不会调用了
 *  sourceApplication 是当开当前应用程序的源程序的BundleId
 */
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    NSString *urlStr = url.absoluteString;
    NSLog(@"%@--%@",urlStr,sourceApplication);
    // 从url中截取字符串,获取地址
    UINavigationController *nav = (UINavigationController *)self.window.rootViewController;
    [nav popToRootViewControllerAnimated:NO]; // 每次弹出前都回到主界面
    ViewController *vc = [nav.childViewControllers firstObject]; // 取出第一个控制器
    vc.urlString = urlStr;
    if ([urlStr containsString:@"public"]) {
        [vc performSegueWithIdentifier:@"public" sender:nil];
    }
    else if([urlStr containsString:@"private"]){
        [vc performSegueWithIdentifier:@"private" sender:nil];
    }
    return YES;
}
```
	- 控制器的跳转实现方法`prepareForSegue`

```objc
// 控制器跳转之前会调用这个方法
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
    if ([segue.identifier isEqualToString:@"public"]) {
        SLQPublicViewController *publicVC = segue.destinationViewController;
        publicVC.urlString = self.urlString;
    }
}
```
- 回调
	- 在源程序调用之前，将自己的URL通过URL传递到目的控制器,格式可以自己定义
	`[self openAppWithString:@"yueyue://public?momo"];
 [self openAppWithString:@"yueyue://private?momo"];`
 	- 然后在目的控制器实现一个返回的方法即可

```objc
/**
 *	回到源控制器
 */
- (IBAction)backToSourceApp:(id)sender {
    // 1、获取源程序的URL
    NSString *urlstr =  [[[self.urlString componentsSeparatedByString:@"?"] lastObject] stringByAppendingString:@"://"];
      NSURL *url = [NSURL URLWithString:urlstr];
    // 2、先判断是否可以打开
    if ([[UIApplication sharedApplication] canOpenURL:url]) {
        // 3、打开URL
        [[UIApplication sharedApplication] openURL:url];
    }
}
```

## 社交分享

### iOS自带

- 支持种类比较少
- iOS自带：Social.framework
- 前提是在iOS系统中配置一下账号

![](../images/Snip20150813_17.png)

- 可以直接添加一些文字、图片、URL等

![](../images/Snip20150813_21.png)

- 如果想监听按键的状态
![](../images/Snip20150813_24.png)


```objc
- (IBAction)share {

    // 1、判断是否已经登陆
    if(![SLComposeViewController isAvailableForServiceType:SLServiceTypeSinaWeibo])
    {
        NSLog(@"平台不可用,或者没有配置相关的帐号");
        return;
    }

    // 2、创建分享控制器
    SLComposeViewController *share = [SLComposeViewController  composeViewControllerForServiceType:SLServiceTypeSinaWeibo];
    // 设置分享文字
    [share setInitialText:@"iOS系统自带社交分享测试！！"];
    // 设置分享图片
    [share addImage:[UIImage imageNamed:@"010"]];
    // 设置分享链接，这里的网址会直接跟在文字后面，不能直接点击
    [share addURL:[NSURL URLWithString:@"www.google.com"]];
    // 3、弹出控制器
    [self presentViewController:share animated:YES completion:nil];
    // 4、监听按钮点击
    share.completionHandler = ^(SLComposeViewControllerResult result)
    {
        // 这里只有两个按钮，一个取消，一个完成
        if(result == SLComposeViewControllerResultCancelled)
        {
            NSLog(@"取消");
        }
        else
        {
            NSLog(@"发表完成");
        }
    };
}
```

- 第三方
	- 友盟
	- ShareSDK
	- 百度分享

### 第三方

- 第三方SDK-静态库

#### 友盟
- 官方帮助文档
<http://dev.umeng.com/social/ios/quick-integration>

- 直接在官网下载sdk，注册，获取appKey
- 在Xcode里实现即可

```objc
// 在代理中实现
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    [UMSocialData setAppKey:@"55cc8af067e58e6e2a0027e0"];
    return YES;
}
// 在控制器中实现
- (IBAction)youmeng:(id)sender {
    [UMSocialSnsService presentSnsIconSheetView:self
                                         appKey:@"55cc8af067e58e6e2a0027e0"
                                      shareText:@"这只是一个测试而已"
                                     shareImage:[UIImage imageNamed:@"xingxing"]
                                shareToSnsNames:[NSArray arrayWithObjects:UMShareToSina,UMShareToTencent,UMShareToRenren,nil]
                                       delegate:nil];
}
```
- 登陆界面，也可以自定义界面
![](../images/Snip20150813_41.png)

-  授权
	- oaauth 2.0 - 弹出一个webView
	URL写`weibo://oaauth`，直接进入此界面

	- SSO - 按照友盟的详细步骤

### 推送通知

- 本地推送通知
![](../images/Snip20150814_45.png)

![](../images/Snip20150813_28.png)

- iOS8需要进行注册
```objc
    // iOS8进行本地通知的话，需要提前注册
    if([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0){
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge | UIUserNotificationTypeAlert | UIUserNotificationTypeSound categories:nil];
        // 注册
        [application registerUserNotificationSettings:settings];
    }
```
![](../images/Snip20150813_31.png)

![](../images/Snip20150814_43.png)

![](../images/Snip20150814_42.png)

- 第一次运行程序时会弹出提醒用户是否接收通知

![](../images/Snip20150813_27.png)

- iOS7会自己弹出
- alertView

![](../images/Snip20150813_29.png)
- 其他属性
![](../images/Snip20150813_32.png)

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

![](../images/Snip20150814_44.png)

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

![](../images/Snip20150814_46.png)

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

![](../images/Snip20150813_35.png)

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

![](../images/Snip20150813_40.png)

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

- 第三方apns
	- JPush
	- <https://www.jpush.cn>
