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


## 导航控制器跨级跳转


```objc
//
//  ViewController.m
//  SLQScrollView
//
//  Created by songlq on 15/10/27.
//  Copyright (c) 2015年 songlq. All rights reserved.
//

#import "ViewController.h"
#import "SLQScrollView.h"
#import "ChatpanelViewController.h"

@interface ViewController () <ChatpanelViewControllerDelegate>

@property (nonatomic,strong) CADisplayLink *displayLink;
@property (nonatomic,strong) NSTimer *timer;
@property (nonatomic, weak) ChatpanelViewController *currentVC;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    SLQScrollView *fatherView = [[SLQScrollView alloc] initWithFrame:CGRectMake(0, 0, 300, 300)];
    fatherView.backgroundColor = [UIColor redColor];
    
    UIButton *btn = [[UIButton alloc] initWithFrame:CGRectMake(100, 100, 100, 44)];
    btn.backgroundColor = [UIColor grayColor];
    [btn setTitle:@"点击" forState:UIControlStateNormal];
    [btn addTarget:self action:@selector(btnClick:) forControlEvents:UIControlEventTouchUpInside];
    btn.tag = 100 + 1;
//    btn.enabled = NO;
//    btn.userInteractionEnabled = NO;
    [fatherView addSubview:btn];
    [self.view addSubview:fatherView];
    

}
- (void)btnClick:(UIButton *)btn
{
    NSLog(@"btnClick");
    ChatpanelViewController *vc = [[ChatpanelViewController alloc] init];
//    self.currentVC = vc;
    vc.delegate = self;
    [self.navigationController pushViewController:vc animated:YES];
}

- (void)chatpanelViewControllerDidClickPushBtn:(ChatpanelViewController *)newVc
{
    [self.navigationController popViewControllerAnimated:YES];
    newVc.delegate = self;
    [self.navigationController pushViewController:newVc animated:YES];
}
//- (void)chatpanelViewControllerDidClickPushBtn:(ChatpanelViewController *)newVc andPreviousVC:(ChatpanelViewController *)preVC
//{
//    [self.navigationController popViewControllerAnimated:YES];
//    newVc.delegate = self;
//    [self.navigationController pushViewController:newVc animated:YES];
////    self.currentVC = newVc;
//}

@end
```