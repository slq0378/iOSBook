## 关于按钮多次点击问题

- 在处理点击事件或者其他类似事件时，要避免多次点击并没有很好的方法

### 下面这一种我认为比较好的方式

- 通过performSelector 延迟某方法的执行，然后再通过cancelPreviousPerformRequestsWithTarget:self 来取消之前要执行的方法。
- 这两个方法执行原理就是，performSelector 会将要执行的任务延迟一段时间执行，在延迟的 这段时间内如果执行cancelPreviousPerformRequestsWithTarget:self方法就会取消这个任务，然后开始新的任务。

```objc
// 代理事件
- (void)pushOperationPanel:(long long)userId
{
    NSNumber *currentUserId = [NSNumber numberWithLongLong:userId];
    [[self class] cancelPreviousPerformRequestsWithTarget:self selector:@selector(delaySendRequest:) object:currentUserId]; // 取消之前添加的任务
    [self performSelector:@selector(delaySendRequest:) withObject:currentUserId afterDelay:0.4]; // 添加新任务
}

- (void)delaySendRequest:(NSNumber *)num
{
    long long userId = num.longLongValue;
    NSLog(@"hahaha");
}
```

### 如果真是按钮的话，倒是好操作，也可以添加定时器

```objc
// 代理事件
- (void)pushOperationPanel:(UIButton *)btn
{
    NSNumber *currentUserId = [NSNumber numberWithLongLong:userId];
    btn.enabled = NO;
    //  send request
    [NSTimer scheduledTimerWithTimeInterval:0.4f target:self selector:@selector(resetBtn:) userInfo:nil repeats:NO];
}

- (void)resetBtn
{
    _pushButton.enabled = YES;
    NSLog(@"hahaha");
}
```

### 还可以通过网络返回数据来决定按钮的状态，如果返回失败就disable，成功就enable，超时就enable。
- 这个应该是比较好的，但是这只是适用于网络请求事件
