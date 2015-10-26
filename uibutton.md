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
