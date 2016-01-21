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
- 


## Exception codes

```
0x8badf00d错误码：Watchdog超时，意为“ate bad food”。
0xdeadfa11错误码：用户强制退出，意为“dead fall”。
0xbaaaaaad错误码：用户按住Home键和音量键，获取当前内存状态，不代表崩溃。
0xbad22222错误码：VoIP应用（因为太频繁？）被iOS干掉。
0xc00010ff错误码：因为太烫了被干掉，意为“cool off”。
0xdead10cc错误码：因为在后台时仍然占据系统资源（比如通讯录）被干掉，意为“dead lock”。
```
## Exception types

```
查看我们的crash分析报告邮件，会发现最经常遇到的错误类型是SEGV（Segmentation Violation，段违例），表明内存操作不当，比如访问一个没有权限的内存地址。
当我们收到SIGSEGV信号时，可以往以下几个方面考虑：
访问无效内存地址，比如访问Zombie对象；
尝试往只读区域写数据；
解引用空指针；
使用未初始化的指针；
栈溢出；
此外，还有其它常见信号：
SIGABRT：收到Abort信号，可能自身调用abort()或者收到外部发送过来的信号；
SIGBUS：总线错误。与SIGSEGV不同的是，SIGSEGV访问的是无效地址（比如虚存映射不到物理内存），而SIGBUS访问的是有效地址，但总线访问异常（比如地址对齐问题）；
SIGILL：尝试执行非法的指令，可能不被识别或者没有权限；
SIGFPE：Floating Point Error，数学计算相关问题（可能不限于浮点计算），比如除零操作；
SIGPIPE：管道另一端没有进程接手数据；
```