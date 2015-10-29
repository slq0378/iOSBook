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

![](images/屏幕快照 2015-10-28 下午10.04.11.png)

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

## UIButton 默认布局
- 默认布局是横排：图片在左，label在右，并且居中方式是水平居中对齐，垂直居中对齐。
    - ![](images/屏幕快照 2015-10-29 08.44.43.png)
    - 紫色是image的frame，绿色是label的frame，黄色是button的frame
- 如果要改变为竖排，那么首先要把对齐方式改为水平靠左，垂直靠上。
    - ![](images/屏幕快照 2015-10-29 08.52.37.png)    
    - 然后再设置每个子控件的`UIEdgeInsets`
    - 最终展示效果是这样的
    - ![](images/屏幕快照 2015-10-29 08.56.50.png)
    - 中间过渡过程是这样的
    - ![](images/屏幕快照 2015-10-29 09.01.23.png)
    - 由此图可见，
``` objc
    [_focusBtn setContentHorizontalAlignment:UIControlContentHorizontalAlignmentLeft];
    [_focusBtn setContentVerticalAlignment:UIControlContentVerticalAlignmentTop];
```
    
- 注：在iOS中，UIEdgeInsets跟web中的padding一样的分top，left，bottom和right，正数就表示与周围的距离变大，例如：默认在是0，我把top变成了10，这时该元素与顶端的距离就会增加10，即元素下移，距离变大。如果是负数的话效果就相反了，距离拉近。
