# 圆形头像

![](../images/屏幕快照 2015-08-07 17.18.19.png)

![](../images/屏幕快照 2015-08-07 17.18.10.png)

- layer比较卡

```objc
// 显示圆形头像方式1：直接修改图片的layer
// 有点卡。不太好
self.iconImageView.layer.cornerRadius = self.iconImageView.width * 0.5;
self.iconImageView.layer.masksToBounds = YES;
```
- 绘图实现
	- 透明
```objc
@implementation UIImage (SLQCircleImage)
/**
 * 返回圆形图片
 */
- (instancetype)circleImage
{
    // 开启图形上下文,带有透明背景的上下文
    UIGraphicsBeginImageContextWithOptions(self.size, NO, 0.0);
    // 建立圆形裁切
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    //  绘制一个圆
    CGRect rect = CGRectMake(0, 0, self.size.width, self.size.height);
    CGContextAddEllipseInRect(ctx, rect);
    // 裁剪
    CGContextClip(ctx);
    // 绘制图片到上下文
    [self drawInRect:rect];
    // 从图形上下文获取图片
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    // 关闭图形上下文
    UIGraphicsEndImageContext();
    return image;
}
@end
```
	- 头像url可能返回nil值，判断一下（可将下载图片方式封装起来UIImageView）

```objc
@implementation UIImageView (SLQCircleImage)
/**
 *	通过URL获取图片，并对图片进行处理，返回圆形图片
 */
- (void) setHeaderWithUrl:(NSString *)url
{
    UIImage *placeholder = [[UIImage imageNamed:@"defaultUserIcon"] circleImage];
    [self sd_setImageWithURL:[NSURL URLWithString:url] placeholderImage:placeholder completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
        // 如果image为空就显示占位图片
        self.image = image ? [image circleImage] : placeholder ;
    }];
}
@end
```
