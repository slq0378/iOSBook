# 连续点击问题

```objc
//点击@xxxx弹出操作面板
- (void)pushOperationPanel:(long long)userId
{
    _isClickUser = YES;
    NSNumber *currentUserId = [NSNumber numberWithLongLong:userId];
    [[self class] cancelPreviousPerformRequestsWithTarget:self selector:@selector(delaySendRequest:) object:currentUserId];
    [self performSelector:@selector(delaySendRequest:) withObject:currentUserId afterDelay:0.4];
}

- (void)delaySendRequest:(NSNumber *)num
{
    long long userId = num.longLongValue;
    [[_facade getChatbarProxy] operationChatbarWithUserid:userId chatbarId:_chatbarInfo.chatbarId];
}

```

