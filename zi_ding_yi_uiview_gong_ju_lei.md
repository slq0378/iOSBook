# 自定义UIView工具类
## 自定义UIView工具类
- 因为要经常获取控件的frame、height等属性进行设置，这里对UIView写一个分类，添加一些自定义的setter和getter方法
- 这样做得依据是
    - 分类中声明@property, 只会生成方法的声明, 不会生成方法的实现和带有_下划线的成员变量
    - 所以可以直接写几个关于这个常用属性的方法

```objc
#import <UIKit/UIKit.h>

@interface UIView (SLQExtension)
/** 在分类中声明@property, 只会生成方法的声明, 不会生成方法的实现和带有_下划线的成员变量*/

/**height*/
@property (nonatomic, assign) CGFloat height;
/**width*/
@property (nonatomic, assign) CGFloat width;
/**size*/
@property (nonatomic, assign) CGSize size;
/**x*/
@property (nonatomic, assign) CGFloat x;
/**y*/
@property (nonatomic, assign) CGFloat y;

@end

//#import "UIView+SLQExtension.h"

@implementation UIView (SLQExtension)

- (void)setWidth:(CGFloat)width
{
    CGRect rect = self.frame;
    rect.size.width = width;
    self.frame = rect;
}

- (CGFloat)width
{
    return self.frame.size.width;
}

- (void)setHeight:(CGFloat)height
{
    CGRect rect = self.frame;
    rect.size.height = height;
    self.frame = rect;
}

- (CGFloat)height
{
    return self.frame.size.height;
}

- (void)setSize:(CGSize)size
{
    CGRect rect = self.frame;
    rect.size = size;
    self.frame = rect;
}

- (CGSize)size
{
    return self.frame.size;
}

- (void)setY:(CGFloat)y
{
    CGRect rect = self.frame;
    rect.origin.y = y;
    self.frame = rect;
}

- (CGFloat)y
{
    return self.frame.origin.y;
}

- (void)setX:(CGFloat)x
{
    CGRect rect = self.frame;
    rect.origin.x = x;
    self.frame = rect;
}

- (CGFloat)x
{
    return self.frame.origin.x;
}

@end
```
