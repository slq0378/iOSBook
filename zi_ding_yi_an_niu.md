# 自定义按钮

## 自定义按钮

- 文字在下，图片在上
- 这个改变任意，只需在layoutSubviews里改变frame即可

```objc
//#import "SLQVerticalButton.h"

@implementation SLQVerticalButton

- (void)setup
{
    self.titleLabel.textAlignment = NSTextAlignmentCenter;
}
/**
 *	通过xib创建按钮
 */
- (void)awakeFromNib
{
    [self setup];
}
/**
 *	通过代码创建按钮
 */
- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        [self setup];
    }
    return self;
}
/**
 *	重新布局子控件
 */
- (void)layoutSubviews
{
    [super layoutSubviews];
    // 调整imageView位置
    self.imageView.x = 0;
    self.imageView.y = 0;
    self.imageView.width = self.width;
    self.imageView.height = self.imageView.width;

    // 调整lable位置
    self.titleLabel.x = 0;
    self.titleLabel.y = self.imageView.height;
    self.titleLabel.width = self.width;
    self.titleLabel.height = self.height - self.imageView.height;
}
@end
```
