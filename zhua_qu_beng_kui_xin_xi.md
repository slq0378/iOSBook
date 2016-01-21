# 抓取崩溃信息

- 如果不能用第三方的，只能自己抓取，上传到自己的服务器

- 在appDelegate里做初始化

```objc
    // 加入crash handler
    NSSetUncaughtExceptionHandler(&uncaughtExceptionHandler);
    
    // 模拟错误信息
//    NSArray *array = [NSArray arrayWithObject:@"0"];
//    NSLog(@"%@", [array objectAtIndex:1]);

```

```objc
/**
 * 实现NSUncaughtExceptionHandler方法
 */
void uncaughtExceptionHandler(NSException *exception)
{
    // 调用堆栈
    NSArray *arr = [exception callStackSymbols];
    // 错误reason
    NSString *reason = [exception reason];
    // exception name
    NSString *name = [exception name];
    
    // 根据自己的需求将crash信息记录下来，下次启动的时候传给服务器。
    // 尽量不要在此处将crash信息上传，因为App将要退出，不保证能够将信息上传至服务器
    
    [[NSUserDefaults standardUserDefaults] setObject:@"YES" forKey:@"isException"];
    [[NSUserDefaults standardUserDefaults] setObject:arr forKey:@"callStackSymbols"];
    [[NSUserDefaults standardUserDefaults] setObject:reason forKey:@"reason"];
    [[NSUserDefaults standardUserDefaults] setObject:name forKey:@"name"];
    
}
```

- 然后在合适的时间把这个信息上传到服务器，上传成功后改`isException` 为`YES`