# 自定义UIWindow显示状态栏通知信息

```objc
//#import "SLQStatusBar.h"
//#define SLQScreenW [UIScreen mainScreen].bounds.size.width
//#define SLQWindowH (20)
//#define SLQStatusFontSize [UIFont systemFontOfSize:12]
@implementation SLQStatusBar

static UIWindow *window_;
static NSTimer *timer_;
+ (void)show
{
    [self showWindow];
}

+ (void)showWindow
{
    CGRect frame = CGRectMake(0,  - SLQWindowH, SLQScreenW, SLQWindowH);
    window_ = [[UIWindow alloc] init];
    window_.frame = frame;
    window_.windowLevel = UIWindowLevelAlert;
    window_.hidden = NO;
    window_.backgroundColor = [UIColor blackColor];
    frame.origin.y = 0;
    [UIView animateWithDuration:0.1 animations:^{
        window_.frame = frame;
    }];
}

+ (void)showMessage:(NSString *)msg
{
    [self showMessage:msg image:nil];
}

+ (void)showMessage:(NSString *)msg image:(UIImage *)image
{
    // 停止计时器
    [timer_ invalidate];


    [self showWindow];
    // 使用按钮显示
    UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
    btn.frame = window_.bounds;
    btn.titleLabel.font = SLQStatusFontSize;
    [btn setTitle:msg forState:UIControlStateNormal];
    if (image != nil) {
        [btn setImage:image forState:UIControlStateNormal];
        btn.titleEdgeInsets = UIEdgeInsetsMake(0, 10, 0, 0);
    }
    [window_ addSubview:btn];
    // 添加定时器
    timer_ = [NSTimer scheduledTimerWithTimeInterval:2 target:self selector:@selector(hide) userInfo:nil repeats:NO];
}

+ (void)showSuccess:(NSString *)msg
{
    [self showMessage:msg image:[UIImage imageNamed:@"SLQStatusBar.bundle/success"]];
}

+ (void)showError:(NSString *)msg
{
    [self showMessage:msg image:[UIImage imageNamed:@"SLQStatusBar.bundle/error"]];
}

+ (void)showLoading:(NSString *)msg
{
    [self showWindow];

    // 显示文字到窗口
    UILabel *lable = [[UILabel alloc] init];
    lable.text = msg;
    lable.textColor = [UIColor whiteColor];
    lable.textAlignment = NSTextAlignmentCenter;
    lable.font = SLQStatusFontSize;
    lable.frame = window_.bounds;

    [window_ addSubview:lable];

    // 显示菊花转动图标
    UIActivityIndicatorView *acti = [[UIActivityIndicatorView alloc] initWithActivityIndicatorStyle:UIActivityIndicatorViewStyleWhite];
    [acti startAnimating];

    // 文字宽度
    CGFloat msgW = [msg sizeWithAttributes:@{NSFontAttributeName : SLQStatusFontSize}].width;
    acti.center = CGPointMake( (SLQScreenW  - msgW) * 0.5 - 20, SLQWindowH * 0.5);
   [window_ addSubview:acti];

}

+ (void)hide
{
    // 停止计时器
    [timer_ invalidate];
    // 销毁计时器
    timer_ = nil;

    CGRect frame = CGRectMake(0, -20, [UIScreen mainScreen].bounds.size.width, 20);

    [UIView animateWithDuration:0.1 animations:^{
        window_.frame = frame;
    } completion:^(BOOL finished) {
        window_ = nil;

    }];
}

@end
```
