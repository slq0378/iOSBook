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