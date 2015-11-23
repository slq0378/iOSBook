# 其他知识

- 判断输入字符串不能为空 

```objc
    //发送文本消息
    NSString *contentString = [text stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]];
    if (![contentString isKindOfClass:[NSString class]] || [contentString isEqualToString:@""])
    {
        BlockAlertView *alertView = [[BlockAlertView alloc] initWithTitle:NSLocalizedString(@"COMMON_ALTER_TITLE", nil)
                                                                  message:NSLocalizedString(@"MESSAGE_INFO_CONTENT_IS_EMPTY", @"内容不能为空")];
        [alertView setCancelButtonWithTitle:NSLocalizedString(@"INFO_BUTTON_KNOW", @"知道了") block:nil];
        [alertView show];
        return;
    }
```

- 短时间内发送过多消息


- 方法1

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


- 方法2

```objc
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

```