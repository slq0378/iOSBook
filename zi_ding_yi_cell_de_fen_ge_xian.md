# 自定义cell的分割线

- 绘制kong的UIView

```objc
- (void)drawRect:(CGRect)rect
{
    
    UIView *line = [[UIView alloc] initWithFrame:CGRectMake(0, rect.size.height - 1, SCREEN_WIDTH, 1)];
    line.backgroundColor = [UIColor blackColor];
    [self addSubview:line];
}
```

- 绘图

