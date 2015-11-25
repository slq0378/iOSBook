# 小问题总结

## 判断用户短时间内发送消息太多
- 看到这个问题时，我想到了定时器，首先定义一个变量`numberOfSend`保存发送次数，然后在第一次输入时开启定时器`timer1`，5s之内如果`numberOfSend`没有超过8条，就在5s后重置`numberOfSend`，如果5s内发送次数超过8条，就开启另一个定时器timer2，来延迟执行发送操作，在timer2中要先取消之前的`timer1`，然后就`return`

```objc

 NSInteger _numberOfSendMessagesInFiveSecond;


- (void)sendMessages
{
    _numberOfSendMessagesInFiveSecond = 0;
}

- (void)sendText:(NSString *)contentString
{
   // 喝杯茶休息一下吧！
    _numberOfSendMessagesInFiveSecond ++ ;
    if ( 1 == _numberOfSendMessagesInFiveSecond){
        
        [self performSelector:@selector(sendMessages) withObject:nil afterDelay:5];
    }
    else if (_numberOfSendMessagesInFiveSecond >= 9) {
        
        [chatInputView insertTextAfterCursor:contentString];
        [self presentSuccessTips:NSLocalizedString(@"CHATPANEL_TAKE_A_CUP_OF_TEA" ,@"喝杯茶休息一下吧！")];
        
        if(_numberOfSendMessagesInFiveSecond == 9) {
            [[self class] cancelPreviousPerformRequestsWithTarget:self selector:@selector(sendMessages) object:nil];
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0f * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                _numberOfSendMessagesInFiveSecond = 0;
            });
        }
        return;
    }
}
```

- 另外一种方式

```objc
@interface P2PSendMsgFrequencyStrategy ()
{
    NSInteger _iMaxCount;
    long long _iMaxMillisecond; //1秒
    NSInteger _iCount;
    NSTimeInterval _lastSendTime;
}

@end

@implementation P2PSendMsgFrequencyStrategy

- (instancetype)init
{
    self = [super init];
    if (self)
    {
        _iMaxCount = 8;
        _iMaxMillisecond = 1*1000; //1秒
        _iCount = 0;
        _lastSendTime = 0;
    }
    return self;
}

- (BOOL)sendMessageTooOften
{//规则是：5秒内发送超过8条就不能发送，然后再过5秒才能发送。
    NSTimeInterval nowSendTime = [[NSDate date]timeIntervalSince1970]*1000;

    long long time = (nowSendTime - _lastSendTime);
    if ((time > _iMaxMillisecond*5 && _iCount < _iMaxCount)
        || (time > _iMaxMillisecond*10 && _iCount >= _iMaxCount)
        )
    {
        _lastSendTime = [[NSDate date]timeIntervalSince1970]*1000;
        _iCount = 0;

        YJLog(@"sendMessage time:%lld", time);
        return NO;
    }
    else
    {
        if (++_iCount < _iMaxCount)
        {
            YJLog(@"sendMessage count: %ld, time:%lld",  (long)_iCount, time);
            return NO;
        }

    }

    YJLog(@"sendMessage Too Many: %ld, time:%lld",  (long)_iCount, time);
    return YES;
}

@end
```