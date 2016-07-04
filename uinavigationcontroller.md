# UINavigationController

- 设置背景图片

```objc
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
- O -> A -> B 变成 O -> B
- 意思就是在通过A中创建B，然后把A pop，再通过导航控制器push B
- 在O中通过代理监听B的创建时刻，将事件传递到O，在O中先pop，然后pushB

- A/B

```objc
//
//  ChatpanelViewController.h
//  SLQScrollView
//
//  Created by Christian on 15/11/12.
//  Copyright © 2015年 songlq. All rights reserved.
//

#import "ViewController.h"

@class ChatpanelViewController;
@protocol ChatpanelViewControllerDelegate <NSObject>

@optional
- (void)chatpanelViewControllerDidClickPushBtn:(ChatpanelViewController *)newVc andPreviousVC:(ChatpanelViewController *)preVC;
- (void)chatpanelViewControllerDidClickPushBtn:(ChatpanelViewController *)newVc;
@end

@interface ChatpanelViewController : ViewController
/***/
@property (nonatomic, weak) id<ChatpanelViewControllerDelegate> delegate;
/**c*/

@end




//
//  ChatpanelViewController.m
//  SLQScrollView
//
//  Created by Christian on 15/11/12.
//  Copyright © 2015年 songlq. All rights reserved.
//

#import "ChatpanelViewController.h"
#import "SLQScrollView.h"

@implementation ChatpanelViewController

- (void)dealloc
{
    NSLog(@"dealloc");
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    
    self.view.backgroundColor = [UIColor colorWithRed:arc4random_uniform(255)/255.0 green:arc4random_uniform(255)/255.0 blue:arc4random_uniform(255)/255.0 alpha:1];
    SLQScrollView *fatherView = [[SLQScrollView alloc] initWithFrame:CGRectMake(0, 0, 300, 300)];
    
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
    if ([self.delegate respondsToSelector:@selector(chatpanelViewControllerDidClickPushBtn:)]) {
        ChatpanelViewController *newVC = [[ChatpanelViewController alloc] init];
        [self.delegate chatpanelViewControllerDidClickPushBtn:newVC];
    }
}

@end
```

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

## 重构导航栈

```objc
NSMutableArray * viewControllers = [self.navigationController.viewControllers mutableCopy];
[viewControllers removeLastObject];
[viewControllers addObject:newController];

[self.navigationController setViewControllers:viewControllers animated:YES];
```