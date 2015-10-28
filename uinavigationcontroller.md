# UINavigationController

- 设置背景图片

```
#pragma mark - 设置导航栏默认背景
- (void)setNavigationBarBackgroundImage
{
    UIImage *img;
    if(IOS7_OR_LATER)
    {
        img = [UIImage imageNamed:@"userinfo_navBg128"];
    }
    else
    {
        img = [UIImage imageNamed:@"userinfo_navBg88"];
    }
    
    UIEdgeInsets edge = UIEdgeInsetsMake(0, 0, 0, 0);
    img = [img resizableImageWithCapInsets:edge resizingMode:UIImageResizingModeTile];
    [self.navigationController.navigationBar setBackgroundImage:img
                                                  forBarMetrics:UIBarMetricsDefault];
}
```