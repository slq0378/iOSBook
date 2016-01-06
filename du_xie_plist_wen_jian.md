# 读写plist文件

- 问题，我有一个plist文件，表示56个民族的，但是里面保存的字典，我想转换成一个数组
- 好的，那么就先遍历这个plist，然后将结果保存到一个数组中，这里出现的一个问题就是C语言字符串转换成NSString的问题，一开始使用`- (nullable id)initWithCString:(const char *)bytes`,一直出问题，转换后有问题。
- 然后我就换了一个方法`- (nullable id)initWithCString:(const char *)bytes length:(NSUInteger)length ` 这个方法转换后没有问题了。
- 第一个plist是按照26个英文字母为key的字典。
- 结果，按照数组保存。

```objc
    // 读取56个民族
    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"nation.plist" ofType:nil];
    NSDictionary *dict2 = [NSDictionary dictionaryWithContentsOfFile:filePath];
    
    // 拼接路径
    NSArray *paths=NSSearchPathForDirectoriesInDomains(NSCachesDirectory,NSUserDomainMask,YES);
    NSString *plistPath1 = [paths objectAtIndex:0];
    NSString *filename = [plistPath1 stringByAppendingPathComponent:@"nation2.plist"];

    NSMutableArray *mutab = [NSMutableArray array];
    
    for (int i = 0 ; i < 26; i ++) {
        char x = 65 + i;
        NSString *str=  [[NSString alloc] initWithCString:&x length:1];
        NSArray *arr = dict2[str];
        for (NSInteger j = 0; j < arr.count; j ++) {
            [mutab addObject:arr[j]];
        }
    }
    // 保存成数组
    [mutab writeToFile:filename atomically:NO];
    NSLog(@"%@",mutab);
```