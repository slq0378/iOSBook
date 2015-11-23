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


```
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