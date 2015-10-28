# UIButton

## UIButton做frame动画时，不响应点击
- 在一个View内部加入几个按钮，然后改变这个view的frame来做动画，但是按钮不响应点击事件。

- 问题代码

```
    __block CGRect rect = _scrollView.frame;
    CGFloat width = [UIScreen mainScreen].bounds.size.width;
    [UIView setAnimationsEnabled:YES];
    [UIView animateWithDuration:0.3f animations:^{
        rect.origin.x -= 10;
        if (rect.origin.x <= - width) {
             // 添加下一个scrollView到self
            [UIView setAnimationsEnabled:NO];
            rect.origin.x = width + 10;
            
        }
        _scrollView.frame = rect;
    } completion:^(BOOL finished) {
        
    }];
```

- 解决问题

```
    [UIView animateWithDuration:0.3f delay:0 options:UIViewAnimationOptionAllowUserInteraction animations:^{
        rect.origin.x -= 10;
        if (rect.origin.x <= - width) {
            // 添加下一个scrollView到self
            [UIView setAnimationsEnabled:NO];
            rect.origin.x = width + 10;
        }
        _scrollView.frame = rect;
    } completion:^(BOOL finished) {
        
    }];
```

- 结论
- 应该是在改变一个控件的frame做动画时，控件的交互被关闭了，所以要在做动画时手动开启交互。

# UIButton 内部布局变化

## 竖排
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    _focusBtn  = [UIButton buttonWithType:UIButtonTypeCustom];
    _focusBtn.frame = CGRectMake(10, 100, 52, 52);
    _focusBtn.backgroundColor = [UIColor greenColor];
//    _focusBtn = [[UIButton alloc] initWithFrame:CGRectMake(70, 70, 52, 52)];
    [_focusBtn setTitle:@"关注" forState:UIControlStateNormal];
    
    CGFloat buttonWidth = 52;
    CGFloat textWidth = 30;
    CGFloat imageWidth = 19;
//    [_focusBtn setImage:[UIImage imageNamed:@"chatpanel_focusBtn"] forState:UIControlStateNormal];
    [_focusBtn setImage:[UIImage imageNamed:@"chatpanel_focusBtn"] forState:UIControlStateNormal];
    [_focusBtn setBackgroundImage:[UIImage imageNamed:@"chatpanel_redcolor"] forState:UIControlStateNormal];
    [_focusBtn setContentHorizontalAlignment:UIControlContentHorizontalAlignmentLeft];
    [_focusBtn setContentVerticalAlignment:UIControlContentVerticalAlignmentTop];
    
    [_focusBtn setImageEdgeInsets:UIEdgeInsetsMake(6,(52-20)/2, 0, _focusBtn.titleLabel.bounds.size.width )];
    [_focusBtn setTitleEdgeInsets:UIEdgeInsetsMake(17.5 + 6 , - ([_focusBtn currentImage].size.width)  + 7 , 9,0 )];
   
    
    [self.view addSubview:_focusBtn];

    
}
```
