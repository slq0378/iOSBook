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