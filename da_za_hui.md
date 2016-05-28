# 大杂烩


- 判断字符串是不是纯数字
```objc
/// 判断字符串是不是纯数字
- (BOOL)isPureNumandCharacters:(NSString *)string
{
    string = [string stringByTrimmingCharactersInSet:[NSCharacterSet decimalDigitCharacterSet]];
    if(string.length > 0)
    {
        return NO;
    }
    return YES;
}
```

- 指定颜色生成图片

```objc
//指定颜色生成图片：
+ (UIImage *)imageWithColor:(UIColor *)color
{
    CGRect rect = CGRectMake(0, 0, 1, 1);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);
    
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return image;
}

```

```objc
//截图
- (UIImage *)grabScreenWithShotView:(UIView *)shotView
{
    UIGraphicsBeginImageContextWithOptions(shotView.bounds.size, NO, [UIScreen mainScreen].scale);
    [shotView drawViewHierarchyInRect:shotView.bounds afterScreenUpdates:YES];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
}
```

## sortedArrayUsingComparator 使用
```objc
    // 数组比较
    NSArray *arr = @[@"able",@"yong",@"song",@"song",@"song",@"sofng",@"sosng",@"song"];
    NSStringCompareOptions comp = NSLiteralSearch ;
    __block NSInteger sameStrCount = 0;
    NSArray *newArr =  [arr sortedArrayUsingComparator:^NSComparisonResult(NSString *obj1, NSString *obj2) {
        NSRange range = NSMakeRange(0, [obj1 length]);
        NSComparisonResult result = [obj1 compare:obj2 options:comp range:range];
        if (result == NSOrderedSame) {
            sameStrCount ++ ;
        }
        return  result;
    }];
    
    [newArr enumerateObjectsUsingBlock:^(NSString *objStr, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"%zd-%@",idx,objStr);
    }];
    NSLog(@"%zd",sameStrCount);
```