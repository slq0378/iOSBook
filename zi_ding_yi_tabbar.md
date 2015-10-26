# 自定义tabBar

## 自定义tabBar

- 系统自带的tabBar不能满足需求
- 自己定义UITabBar
    - 自定义一个类继承自UITabBar
    - 实现initWithFrame和layoutSubviews方法即可。

```objc
//#import "SLQTabBar.h"
@interface SLQTabBar ()
/**发布按钮*/
@property (nonatomic, strong) UIButton *publishBtn;
@end
@implementation SLQTabBar
- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        // 添加一个自定义按钮到tabBar
        UIButton *addBtn = [UIButton buttonWithType:UIButtonTypeCustom];
        [addBtn setBackgroundImage:[UIImage imageNamed:@"tabBar_publish_icon"] forState:UIControlStateNormal];
        [addBtn setBackgroundImage:[UIImage imageNamed:@"tabBar_publish_click_icon"] forState:UIControlStateHighlighted];

        [self addSubview:addBtn];
        self.publishBtn = addBtn;
    }
    return self;
}

- (void)layoutSubviews
{
    [super layoutSubviews];

    // 设置发布按钮的frame
    self.publishBtn.frame =CGRectMake(0, 0, self.publishBtn.currentBackgroundImage.size.width, self.publishBtn.currentBackgroundImage.size.height);
    self.publishBtn.center = CGPointMake(self.frame.size.width * 0.5, self.frame.size.height * 0.5);

    // 设置其他item的位置
    CGFloat x = 0.0;
    CGFloat y = 0;
    CGFloat width = self.frame.size.width / 5;
    CGFloat height = self.frame.size.height;

    NSInteger index = 0;
    for (UIView *btn in self.subviews) {
        // 判断按钮属性
        if(![btn isKindOfClass:NSClassFromString(@"UITabBarButton")] || (btn == self.publishBtn))
//        if (![btn isKindOfClass:[UIControl class]] || (btn == self.publishBtn))
        {
            continue;
        }
        // 计算x坐标
        x = ((index > 1)?(index + 1):index ) * width; // 跳过加号按钮，然后继续设置frame
        btn.frame = CGRectMake(x, y, width, height);

        // 索引增加
        index ++;
    }
}
```


![](/images/屏幕快照 2015-07-22 21.00.27.png)

- 哈哈
