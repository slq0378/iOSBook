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

```objc
- (void)drawRect:(CGRect)rect
{
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    CGContextSetFillColorWithColor(context, [UIColor clearColor].CGColor);
    CGContextFillRect(context, rect);
    
    上分割线，
    CGContextSetStrokeColorWithColor(context, [UIColor blackColor].CGColor);
    CGContextStrokeRect(context, CGRectMake(5, -1, rect.size.width - 10, 1));
    
    //下分割线
    CGContextSetStrokeColorWithColor(context, [UIColor blackColor].CGColor);
    CGContextStrokeRect(context, CGRectMake(5, rect.size.height, rect.size.width - 10, 1));
}
```