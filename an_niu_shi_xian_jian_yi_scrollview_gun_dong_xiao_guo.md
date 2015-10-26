# 按钮实现简易scrollView滚动效果

- 按钮添加指示器以及实现滚动view效果

![](../images/按钮添加指示器效果.gif)

```objc
/**
 *	设置顶部的标签栏
 */
- (void)setTitlesView
{
    // 一个UIView + 5个UIButton
    UIView *titleView = [[UIView alloc] init];
    titleView.backgroundColor = SLQRGBColor(241, 241, 241);
    // 指定尺寸和位置
    titleView.frame = CGRectMake(0, 64, self.view.width, 44);
    NSArray *title = @[@"全部",@"视频",@"声音",@"图片",@"段子"];
    // 添加5个按钮
    CGFloat x = 0;
    CGFloat width = titleView.width / title.count;
    for (NSInteger i = 0 ; i < 5;  i ++) {
        UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
        x = i * width;
        btn.frame = CGRectMake(x, 0, width, titleView.height);
        [btn setTitle:title[i] forState:UIControlStateNormal];
        [btn setTitleColor:[UIColor grayColor] forState:UIControlStateNormal];
        [btn addTarget:self action:@selector(titleClick:) forControlEvents:UIControlEventTouchUpInside];
        [titleView addSubview:btn];
    }

    // 添加底部指示条indicator
    UIView *indicatorView = [[UIView alloc] init];
    indicatorView.backgroundColor = [UIColor redColor];
    // 位置动态计算,添加到按钮底部
    indicatorView.height = 2;
    indicatorView.y = titleView.height - indicatorView.height;

    [titleView addSubview:indicatorView];

    self.indicatorView = indicatorView;
    [self.view addSubview:titleView];
}
/**
 *	点击按钮首先切换指示器的位置
 */
- (void)titleClick:(UIButton *)btn
{
    [UIView animateWithDuration:0.1 animations:^{
        // 设置位置
        self.indicatorView.width = btn.titleLabel.width;
        self.indicatorView.centerX = btn.centerX;
    }];
}
```
