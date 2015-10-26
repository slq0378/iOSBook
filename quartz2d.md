# Quartz2D-绘图

## 简介
- Quartz 2D是一个二维绘图引擎，同时支持iOS和Mac系统
- Quartz 2D能完成的工作
    - 绘制图形 : 线条\三角形\矩形\圆\弧等
    - 绘制文字
    - 绘制\生成图片(图像)
    - 自定义UI控件
- 使用Quartz 2D绘制图形需要绘制在UIView上，而且要自定义的view。

- 自定义view的步骤
    - 1、新建一个类，继承自UIView
    - 2、实现- (void)drawRect:(CGRect)rect方法，然后在这个方法中
    - 3、取得跟当前view相关联的图形上下文
    - 4、绘制相应的图形内容
    - 5、利用图形上下文将绘制的所有内容渲染显示到view上面

## 几种简单的绘图方式

- 绘图方式1

```objc

    #pragma mark - 最原始的绘图方式1
    - (void)draw2 {
        // Drawing code
        //获得图形上下文
        CGContextRef con = UIGraphicsGetCurrentContext();
        // 创建路径
        CGMutablePathRef path = CGPathCreateMutable();
        // 绘制一条线
        //    CGPathMoveToPoint(path, NULL, 50, 50);
        //    CGPathAddLineToPoint(path, NULL, 200, 200);
        // 绘制矩形
        //    CGPathAddRect(path, NULL, CGRectMake(60, 60, 100, 100));
        //    CGPathAddEllipseInRect(path, NULL, CGRectMake(60, 60, 100, 100));
        // 圆角矩形
        //    CGPathAddRoundedRect(path, NULL, CGRectMake(60, 60, 100, 100), 5, 5);
        // 弧线
        CGPathAddArc(path, NULL, 100, 100,30, 0, M_PI, YES);
        // 添加路径到上下文中
        CGContextAddPath(con, path);
        // 显示到view中
        CGContextStrokePath(con);
    }
```

- 绘图方式2

```objc
    #pragma mark - 绘图方式2
    - (void)draw1 {
        // Drawing code
        //获得图形上下文
        CGContextRef con = UIGraphicsGetCurrentContext();
        // 绘制路径
        // 绘制直线
        //    CGContextMoveToPoint(con, 0, 0);
        //    CGContextAddLineToPoint(con, 100, 100);
        //    CGContextAddLineToPoint(con, 50, 100);
        //CGContextMoveToPoint(con, 50, 50);
        //
        // 绘制圆
        //    CGContextAddEllipseInRect(con, CGRectMake(60, 60, 100, 100));
        // 绘制椭圆
        //    CGContextAddEllipseInRect(con, CGRectMake(60, 60, 150, 100));
        // 绘制矩形
        //    CGContextAddRect(con, CGRectMake(60, 60, 150, 100));
        // 绘制
        CGContextAddArc(con, 0, 0, 50, M_PI, M_PI_2, YES);
        // 显示到view中
        CGContextStrokePath(con);
```

- 绘图方式3

```objc
    #pragma mark - 绘图方式3-贝瑟尔路径绘图
    // 贝瑟尔路径绘图
    - (void)draw3 {
        // Drawing code
        // UIKit已经封装了一些绘图的功能
        // 贝瑟尔路径
        UIBezierPath *path = [UIBezierPathbezierPath];
        // 绘制路径
        //    [path moveToPoint:CGPointMake(100, 100)];
        //    [path addLineToPoint:CGPointMake(200, 200)];
        // 圆弧
        [path addArcWithCenter:CGPointMake(100, 100) radius:50startAngle:0endAngle:M_PIclockwise:YES];// 顺时针绘制一个弧线
        [path addLineToPoint:CGPointMake(100, 100)];
        [[UIColorredColor] setStroke]; // 设置线条颜色
        //
        path.lineJoinStyle = kCGLineJoinRound; //
        path.lineWidth = 2; // 宽度
        path.lineCapStyle = kCGLineCapRound; // 样式
        [path fill];
        // 显示
        [path stroke];
    }
```
- 绘图状态的设置

```objc
    #pragma mark - 设置线条状态在渲染之前
    - (void)draw4 {
        // Drawing code
        // 获得图形上下文
        CGContextRef ct = UIGraphicsGetCurrentContext();
        // 绘制路径
        CGContextMoveToPoint(ct, 100, 100);
        CGContextAddLineToPoint(ct, 200, 200);
        CGContextAddLineToPoint(ct, 100, 300);
        // 设置绘图状态，一定要在渲染之前设置，并且一经设置，状态会一直持续下去，除非再次改变。
        CGContextSetLineWidth(ct, 5);
        [[UIColorredColor] setStroke];
        CGContextSetLineJoin(ct, kCGLineJoinRound);
        CGContextSetLineCap(ct, kCGLineCapRound);
        // 渲染
        CGContextStrokePath(ct);
    }
```

- 绘制包含多个状态的图形

```objc
    - (void)drawRect:(CGRect)rect {
        // Drawing code
        // 绘制多个状态不同的线
        // 获得图形上下文
        UIBezierPath *path = [UIBezierPath bezierPath];
        // 绘制路径
        [path moveToPoint:CGPointMake(100, 100)];
        [path addLineToPoint:CGPointMake(200, 200)];
        // 设置绘图状态，一定要在渲染之前设置，并且一经设置，状态会一直持续下去，除非再次改变。
        [[UIColor redColor] setStroke];
        // 渲染
        [path stroke];
        // 获得图形上下文
        UIBezierPath *path1 = [UIBezierPath bezierPath];
        // 绘制路径
        [path1 moveToPoint:CGPointMake(200 , 200)];
        [path1 addLineToPoint:CGPointMake(100, 150)];
        // 设置绘图状态，一定要在渲染之前设置，并且一经设置，状态会一直持续下去，除非再次改变。
        [[UIColor blueColor] setStroke];
        // 渲染
        [path1 stroke];
    }
```

##绘制饼状图

- 首先花圆弧，然后连接圆心，最后填充位一个扇形

  ![](../images/屏幕快照 2015-06-21 12.44.58.png)

```objc
    // 饼状图2
    - (void)drawRect:(CGRect)rect{
        NSArray *arr = [self randomArray]; // 随机地返回一个数组，数组元素和为100
        // 获取半径和圆心
        CGFloat radius = self.frame.size.height/2 - 2;
        CGPoint center = CGPointMake(radius, radius);
        // 绘制角度
        CGFloat startAngle = 0;
        CGFloat endAngle = 0;
        // 绘制图形
        for (int i = 0 ; i < arr.count; i ++) {
            // 计算角度
            endAngle = startAngle + [arr[i] floatValue] / 100.0 * M_PI * 2;
            // 绘制图形
            UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center radius:radius startAngle:startAngle endAngle:endAngle clockwise:YES];
            [path addLineToPoint:center];
            startAngle = endAngle;
            // 设置颜色
            [[self randomColor] set];
            // 填充，并且把终点和起始点连接起来
            [path fill];
        }
    }
```

- **随机返回数组，且元素和为100**

```objc
    // 返回随机数组，且数组元素和为100
    - (NSArray *)randomArray
    {
        NSMutableArray *arr = [NSMutableArrayarray];
        int total = 100;
        int temp = 0;
        for (int i = 0 ; i < arc4random_uniform(10) + 1;i ++) {
            // 100 1~100
            temp = arc4random_uniform(total) + 1;
            // 随机出来的临时值等于总值，直接退出循环，因为已经把总数分配完毕，没必要在分配。
            [arr addObject:@(temp)];
             // 解决方式：当随机出来的数等于总数直接退出循环。
            if (temp == total) {
                break;
            }
            total -= temp;
        }
        // 如果总数大于0就添加到数组中
        if (total ) {
            [arr addObject:@(total)];
        }
        return  arr;
    }
```
- 返回随机颜色

```objc
    // 返回随机的颜色
    - (UIColor *)randomColor
    {
        // iOS：RGB返回是0~1
        CGFloat r = arc4random_uniform(256) / 255.0;
        CGFloat g = arc4random_uniform(256) / 255.0;
        CGFloat b = arc4random_uniform(256) / 255.0;
        return [UIColorcolorWithRed:r green:g blue:b alpha:1];
    }
```

##绘制柱状图

- 计算方柱个数，平分宽度，高度按占视图比例计算。

   ![](../images/屏幕快照 2015-06-21 12.54.31.png)

```objc
    // 柱状图
    - (void)drawRect:(CGRect)rect {
        // 随机地返回一个数组，数组元素和为100
        NSArray *arr = [self randomArray];
        // 起始点和高度
        CGFloat x = 0;
        CGFloat y = 0;
        CGFloat w = 0;
        CGFloat h = 0;
        NSInteger count = arr.count;
        // 宽度
        w = rect.size.width / (2*count - 1);
        for (int i = 0 ; i < arr.count; i ++) {
            // x坐标
            x = 2*i * w;
            // 高度
            h = [arr[i] floatValue] / 100.0 * rect.size.height;
            // y坐标
            y = rect.size.height - h;
            UIBezierPath *path = [UIBezierPath bezierPathWithRect:CGRectMake(x, y, w, h)];
            [[self randomColor] set];
            [path fill];
        }
    }
```

## 文本显示

![](../images/屏幕快照 2015-06-21 12.58.54.png)

```objc
    // 文本显示
    - (void)drawRect:(CGRect)rect
    {
        NSString *str = @"哈尽快回家的客户发动机可舒服哈的尽快发货阿红的健康法哈德减肥哈第三方阿姐回复就爱的客户房间卡地方哈就等哈接电话发掘";
        // 这个方法不会自动换行
    //    [str drawAtPoint:CGPointZero withAttributes:nil];
        // 自动换行
        [str drawInRect:rect withAttributes:nil];
    }
```

## 富文本显示

![](../images/屏幕快照 2015-06-21 12.59.51.png)

```objc
    // 富文本：带有状态的文本
    - (void)drawRect:(CGRect)rect
    {
        NSString *str = @"哈尽快回家的客户发动机可舒服哈的尽快发货阿红的健康法哈德减肥哈第三方阿姐回复就爱的客户房间卡地方哈就等哈接电话发掘";
        NSMutableDictionary *dict = [NSMutableDictionarydictionary];
        // 属性的设置可以在UIkit框架的头文件里找到解释
        // 字体颜色
        dict[NSForegroundColorAttributeName] = [UIColorredColor];
        // 字体大小
        dict[NSFontAttributeName] = [UIFontsystemFontOfSize:30];
        // 字体粗细
        dict[NSStrokeWidthAttributeName] = @5;
        // 颜色
        dict[NSStrokeColorAttributeName] = [UIColorgreenColor];
        // 阴影
        NSShadow *sha = [[NSShadow alloc] init];
        sha.shadowOffset = CGSizeMake(5, 5);
        sha.shadowBlurRadius = 10;
        sha.shadowColor = [UIColor yellowColor];
        dict[NSShadowAttributeName] = sha;
        // 绘制到视图
        [str drawInRect:rect withAttributes:dict];
    }
```

## 绘制图片到视图

- drawAtPoint

![](../images/屏幕快照 2015-06-21 13.03.43.png)

- drawInRect

![](../images/屏幕快照 2015-06-21 13.03.26.png)


- drawAsPatternInrect

![](../images/屏幕快照 2015-06-21 13.03.53.png)

- 裁剪

 ![](../images/屏幕快照 2015-06-21 13.04.30.png)

```objc
    // 绘制图形
    - (void)drawRect:(CGRect)rect
    {
        // 超出裁剪区域的内容全部裁剪掉
        // 注意：裁剪必须放在绘制之前
       // UIRectClip(CGRectMake(0, 0, 100, 100));
        UIImage *image = [UIImage imageNamed:@"010"];
        // 默认按照图片比例显示
    //    [image drawAtPoint:CGPointZero];
        // 将整个图片显示到rect中，拉伸或者缩小
        [image drawInRect:rect];
        // 默认填充显示
    //    [image drawAsPatternInRect:rect];
    }
```

- 图形上下文状态

- 保存某个状态到栈顶，用于之后恢复。

```objc
    // 图形上下文3 UIBezierPath:使用[path stroke];时上下文状态有UIBezierPath自身决定
    - (void)drawRect:(CGRect)rect
    {
        //保存上下文状态,如果使用这种方法保存上下文状态的话，需要设置以CGContext开头的那些函数设置状态，
        // 获取图形上下文
        CGContextRef ctx = UIGraphicsGetCurrentContext();
        // 绘制第一条线
        // 贝塞尔路线
        UIBezierPath *path  = [UIBezierPathbezierPath];
        // 绘制路径
        [path moveToPoint:CGPointMake(55, 55)];
        [path addLineToPoint:CGPointMake(99, 90)];
        // 添加路径到上下文
        //将c路径转换成oc对象：CGPath
        CGContextAddPath(ctx, path.CGPath);
        // 保存上下文状态
        CGContextSaveGState(ctx);
        CGContextSetLineWidth(ctx, 5);
        [[UIColorredColor] setStroke];
        // 渲染上下文
    //    [path stroke];
        CGContextStrokePath(ctx);
        // 绘制第二条线
        path  = [UIBezierPath bezierPath];
        // 绘制路径
        [path moveToPoint:CGPointMake(100, 10)];
        [path addLineToPoint:CGPointMake(100, 80)];
        // 恢复上下文状态
        CGContextRestoreGState(ctx);
        // 添加路径到上下文
        CGContextAddPath(ctx, path.CGPath);
    //    [[UIColor blueColor] setStroke];
        // 渲染
    //    [path stroke];
        CGContextStrokePath(ctx);
    }
```

## 绘图刷新-定时器

- 如果在绘图的时候需要用到定时器，通常CADisplayLink
 NSTimer很少用于绘图，因为调度优先级比较低，并不会准时调用

```objc
    // 如果在绘图的时候需要用到定时器，通常CADisplayLink
    // NSTimer很少用于绘图，因为调度优先级比较低，并不会准时调用
    - (void)awakeFromNib
    {
        // 添加计时器，改变_smailY的值
    //    比起NSTimer，CADisplayLink可以确保系统渲染每一帧的时候我们的方法都被调用，从而保证了动画的流畅性。
        // CADisplayLink:每次屏幕刷新的时候就会调用，屏幕一般一秒刷新60次
        CADisplayLink *link = [CADisplayLink displayLinkWithTarget:self selector:@selector(timeChange)];
        // 添加至运行主循环
        [link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];
    }
    - (void)timeChange
    {
        // 注意：这个方法并不会马上调用drawRect,其实这个方法只是给当前控件添加刷新的标记，等下一次屏幕刷新的时候才会调用drawRect
        [self setNeedsDisplay];
    }
```

## 矩阵操作

```objc
    // 上下文矩阵操作
    // 注意:矩阵操作必须要在添加路径之前
    -(void)drawRect:(CGRect)rect
    {
        // 获取图形山下文
        CGContextRef ctx1 = UIGraphicsGetCurrentContext();
        //    CGContextSaveGState(ctx1);
        // 绘制路径1
    //    CGContextMoveToPoint(ctx1, 50, 50);
        CGContextTranslateCTM(ctx1, 100, 0); // 添加路径之前进行矩阵操作
        CGContextScaleCTM(ctx1, 2, 2);
        CGContextRotateCTM(ctx1, M_PI_2);
        CGContextAddEllipseInRect(ctx1, CGRectMake(10, 10, 50, 90));
        [[UIColorredColor] setStroke];
        CGContextSetLineWidth(ctx1, 5);
        CGContextSetLineJoin(ctx1, kCGLineJoinRound);
        // 渲染路径
        CGContextStrokePath(ctx1);
    }
```


## 水印处理

- 给图片添加文字、图片水印

```objc
    // 水印处理
    - (void)shuiyin
    {
        // 水印处理
        UIImage *image  = [UIImage imageNamed:@"4"];
        UIImage *image2  = [UIImage imageNamed:@"010"];

        CGFloat imageH = image.size.height;
        CGFloat imageW = image.size.width;
        // 获取位图上下文
        UIGraphicsBeginImageContextWithOptions(image.size, NO, 0.0);

        // 绘制图形或者文字
        [image drawAtPoint:CGPointZero];
        NSString *str = @"@zhang@163.com";
        CGFloat strlenth = str.length * 8;
        // 文字水印
        [str drawAtPoint:CGPointMake(imageW - strlenth, imageH - 44) withAttributes:nil];
        // 图片水印
        [image2 drawAtPoint:CGPointMake(imageW - strlenth - 66, imageH - 66)];
        // 关闭位图上下文
        UIGraphicsEndPDFContext();
        // 获取位图
        _imageView.image = UIGraphicsGetImageFromCurrentImageContext();
        // 保存图片，需要转换成二进制数据
        NSData *data = UIImageJPEGRepresentation(_imageView.image, 1); // 参数2表示图片质量
        NSData *data1 = UIImagePNGRepresentation(_imageView.image); // png默认最高质量
        // 写入文件
        [data1 writeToFile:@"/Users/song/Desktop/me.png"atomically:YES];
    }
```

![](../images/屏幕快照 2015-06-21 17.44.27.png)


## 图形裁切

- 首先设置一个裁切区域，然后在裁切区域显示图片。

```objc
    // 图形裁切
    - (void)clipImage
    {
        // 获取图片
        UIImage *image = [UIImage imageNamed:@"4"];
        // 获取位图上下文
        UIGraphicsBeginImageContextWithOptions(image.size,NO,0.0);
        // 设置圆形裁剪区域，正切与图片
        UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0, 0, image.size.width, image.size.height)];
        // 添加裁切区域
        [path addClip];
        // 绘制图片
        [ima drawAtPoint:CGPointZero];
        // 从上下文获取图片
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        // 关闭位图上下文
        UIGraphicsEndImageContext();
        // 显示位图
        _imageView.image = image;
    }
```

![](../images/屏幕快照 2015-06-21 18.22.15.png)

- 带边框的圆形裁切
    - 首先绘制一个红色的圆，然后在圆中心绘制图片，保留一定的边框。

```objc
    // 带边框的原型裁切
    - (void)cicleAroundImage
    {
        // 1、绘制一个大圆，设置背景色
        // 获取图片
        UIImage *ima = [UIImage imageNamed:@"4"];
        // 图片宽高
        CGFloat imageW = ima.size.width;
        // 边界宽度
        CGFloat boarder = 3;
        // 圆环宽高
        CGFloat cicleW = imageW + 2 * boarder;
        // 获取位图上下文
        UIGraphicsBeginImageContextWithOptions(CGSizeMake(cicleW, cicleW),NO,0.0);
        // 绘制圆形
        UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0, 0, cicleW, cicleW)];
        // 填充红色
        [[UIColor redColor] set];
        [path fill];
        // 2、绘制图片，要比大圆小一点
        UIBezierPath *clipArea = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(boarder , boarder, imageW, imageW)];
        // 裁切
        [clipArea addClip];
        // 绘制图片
        [ima drawAtPoint:CGPointMake(boarder, boarder)];
        // 从上下文获取图片
         UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
         // 关闭位图上下文
         UIGraphicsEndImageContext();
         // 显示位图
         _imageView.image = image;
    }
```

![](../images/屏幕快照 2015-06-21 20.03.200.png)


- 带有边框的裁切可以使用一个分类

```objc
    #import "UIImage+image.h"
    @implementation UIImage (image)


    + (instancetype) imageWithImage:(UIImage *)image withBoard:(CGFloat )board withColor:(UIColor *)color
    {
        // 1、绘制一个大圆，设置背景色
        // 获取图片
        UIImage *ima = image;
        // 图片宽高
        CGFloat imageW = ima.size.width;
        // 边界宽度
        CGFloat boarder = board;
        // 圆环宽高
        CGFloat cicleW = imageW + 2 * boarder;
        // 获取位图上下文
        UIGraphicsBeginImageContextWithOptions(CGSizeMake(cicleW, cicleW),NO,0.0);
        // 绘制圆形
        UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0, 0, cicleW, cicleW)];
        // 填充红色
        [color set];
        [path fill];


        // 2、绘制图片，要比大圆小一点
        UIBezierPath *clipArea = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(boarder , boarder, imageW, imageW)];
        // 裁切
        [clipArea addClip];
        // 绘制图片
        [ima drawAtPoint:CGPointMake(boarder, boarder)];
        // 从上下文获取图片
        UIImage *clipImage = UIGraphicsGetImageFromCurrentImageContext();
        // 关闭位图上下文
        UIGraphicsEndImageContext();
        // 显示位图
        return clipImage;
    }
    @end
```

- 使用起来也方便，直接包含头文件即可使用

```objc
    #import "ViewController.h"
    #import "UIImage+image.h"


    @interfaceViewController ()
    @property (weak, nonatomic) IBOutletUIImageView *imageView;
    @end


    @implementation ViewController


    - (void)viewDidLoad {
        [super viewDidLoad];
        // 使用分类
        UIImage *ima = [UIImage imageWithImage:[UIImage imageNamed:@"4"] withBoard:3 withColor:[UIColor blueColor]];
        _imageView.image = ima;
    }
```

## 屏幕截图

- 主要是从控制器上获取图层。一定要使用 renderInContext: 进行渲染，不可使用draw...

```objc
    // 屏幕截图
    - (void)snipImage
    {
        //获取位图上下文
        UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, NO, 0);
        // 获取当前上下文
        CGContextRef ctx = UIGraphicsGetCurrentContext();
        // 渲染控制器图层到图形上下文，把控件上的图层渲染到上下文,layer只能渲染
        [self.view.layer renderInContext:ctx];
        // 获取图形上下文
        UIImage *snipImage = UIGraphicsGetImageFromCurrentImageContext();
        // 关闭位图上下文
        UIGraphicsEndImageContext();
        NSData *data = UIImagePNGRepresentation(snipImage);
        [data writeToFile:@"/Users/song/Desktop/snip.png"atomically:YES];
    }
```

## 手势解锁

- 效果
![](../images/手势解锁.gif)

- 4.1、界面
    - 在storyboard里添加一个view用来显示九宫格
![](../images/屏幕快照 2015-06-21 22.37.35.png)

- 4.2、新建一个类继承自UIView
- 添加按钮

```objc
    // 添加按钮
    - (void)awakeFromNib
    {
        [super awakeFromNib];
        // 创建按钮
        for (NSInteger i = 0;  i < 9; i ++)
        {
            UIButton *btn = [UIButtonbuttonWithType:UIButtonTypeCustom];
            [btn setBackgroundImage:[UIImageimageNamed:@"gesture_node_normal"] forState:UIControlStateNormal];
            [btn setBackgroundImage:[UIImageimageNamed:@"gesture_node_highlighted"] forState:UIControlStateSelected];
            btn.userInteractionEnabled = NO;
            btn.tag = i;
            [self addSubview:btn];
        }
    }
```
- 设置按钮frame

```objc
    // 九宫格按钮计算
    - (void)layoutSubviews
    {
        [super layoutSubviews];
        // 按钮个数
        NSInteger count = self.subviews.count;
        // 按钮宽度和高度
        CGFloat x = 0;
        CGFloat y = 0;
        CGFloat w = 74;
        CGFloat h = 74;
        // 行
        NSInteger cols = 3;
        // 列
        NSInteger rows = 3;
        // 间距
        CGFloat margin = (self.bounds.size.width - cols * w)/(cols + 1);
        // 设置frame
        for (NSInteger i = 0;  i < count; i ++)
        {
            UIButton *btn = (UIButton *)self.subviews[i];
            // 当前行
            NSInteger row = i / rows;
            // 当前列
            NSInteger col = i % cols;
            // x
            x = margin + col * (margin + w);
            // y
            y = row * (margin + h);
            btn.frame = CGRectMake(x, y, w, h);
        }
    }
```
- 定义一个可变数组用来保存选中按钮

```objc
    @property (nonatomic,strong) NSMutableArray *selectedBtn;
    - (NSMutableArray *)selectedBtn
    {
        if (_selectedBtn == nil) {
            _selectedBtn = [NSMutableArray array];
        }
        return  _selectedBtn;
    }
```

- 实现滑动手势方法

```objc
    - (IBAction)pan:(UIPanGestureRecognizer *)sender {
    //    NSLog(@"pan");
        // 获取触摸点
        _curP = [sender locationInView:self];
        // 遍历按钮数组
        for (UIButton *btn in self.subviews) {
            // 判断点是否包含在按钮中
            if (CGRectContainsPoint(btn.frame, _curP) && !btn.selected ) {
                btn.selected = YES;
                [self.selectedBtn addObject:btn];
            }
        }
        // 重绘
        [self setNeedsDisplay];
        // 恢复原状
        if (sender.state == UIGestureRecognizerStateEnded ) {
            // 保存密码
            NSMutableString *str = [NSMutableStringstring];
            for (UIButton *btn in self.selectedBtn) {
               [ str appendFormat:@"%ld",btn.tag ];
            }
            NSLog(@"%@",str);
            // 删除选中状态
            [self.selectedBtn makeObjectsPerformSelector:@selector(setSelected:) withObject:@(NO)];
            // 情况选中数组
            [self.selectedBtn removeAllObjects];
        }
    }
```
- 然后在drawRect中进行连线

```objc
    - (void)drawRect:(CGRect)rect
    {
        // 没有选中按钮，直接退出
        if (self.selectedBtn.count == 0) {
            return;
        }
        // 把所有选中的按钮连线
        NSInteger count = self.selectedBtn.count;
        UIBezierPath *path = [UIBezierPathbezierPath];
        // 遍历选中按钮数组，对按钮连线
        for (int i = 0 ; i < count ; i ++)  {
            UIButton *btn = self.selectedBtn[i];
            if (i == 0) {
                // 设置起点
                [path moveToPoint:btn.center];
            }else
            {
                [path addLineToPoint:btn.center];
            }
        }
        // 连接到当前点
        [path addLineToPoint:_curP];
        // 设置连线样式
        [[UIColorgreenColor] set];
        path.lineWidth = 10;
        path.lineJoinStyle = kCGLineJoinRound;
        [path stroke];
    }
```

## 画板

- 5.1、通过自动布局搭建界面


![](../images/屏幕快照 2015-06-22 10.30.13.png)


- 5.2、在自定义view中得实现


![](../images/屏幕快照 2015-06-22 17.21.39.png)

- 使用滑动手势进行画线
- 1、自定义一个继承自UIBezierPath 的类，用来改变线条的颜色，只需添加一个属性即可

```objc
    #import <UIKit/UIKit.h>
    @interface drawPath : UIBezierPath
    @property (nonatomic,strong) UIColor *lineColor;
    @end
```
- 2、自定义一个view封装画板的操作 DrawView

```objc
    #import <UIKit/UIKit.h>
    @classdrawPath;
    @interface DrawView : UIView
    @property (nonatomic, strong) UIColor *lineColor; // 线条颜色
    @property (nonatomic,assign) CGFloat lineWidth; // 线条宽度
    @property (nonatomic,strong) UIImage *image; // 保存加载的图片
    - (void)eraseScreen; // 清屏
    - (void)undo; // 撤销
    @end
```
- 3、类扩展中添加两个属性

```objc
    #import "DrawView.h"
    #import "drawPath.h"
    @interfaceDrawView ()
    @property (nonatomic,strong) drawPath *path; // 绘图路径
    @property (nonatomic,strong) NSMutableArray *drawLines; // 路径数组
    @end


    // 数组懒加载
    - (NSMutableArray *)drawLines
    {
        if (_drawLines == nil) {
            _drawLines = [NSMutableArray array];
        }
        return  _drawLines;
    }
```
- 4、初始化方法

```objc
    - (void)setUp
    {
        // 设置手势
        UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(pan:)];
        [selfaddGestureRecognizer:pan];
        _lineColor = [UIColor blackColor];
        _lineWidth = 1;
    }
```
- 5、手势实现

```objc
    - (void)pan:(UIPanGestureRecognizer *)pan
    {
        // 获取当前点
        CGPoint curPoint = [pan locationInView:self];
        if (pan.state == UIGestureRecognizerStateBegan)
        {
            // 设置起点
            _path = [[drawPath alloc] init];
            _path.lineColor = _lineColor;
            _path.lineWidth = _lineWidth;
            [_path moveToPoint:curPoint];
            // 保存路径
            [self.drawLines addObject:_path];
        }
        // 绘制
        [_path addLineToPoint:curPoint];
        // 重绘
        [self setNeedsDisplay];
    }
```

- 6、重写的构造方法

```objc
    // xib调用
    - (void)awakeFromNib
    {
        [self setUp];
    }
    // 代码调用
    - (instancetype)initWithFrame:(CGRect)frame
    {
        if (self = [super initWithFrame:frame]) {
            [self setUp];
        }
        returnself;
    }
```

- 7、在drawRect 绘制图片

```objc
    // 清空屏幕
    - (void)drawRect:(CGRect)rect
    {
        for (drawPath *path  in self.drawLines) {
            // 判断是不是图片
            if ([path isKindOfClass:[UIImage class]]) {
                // 绘制图片
                UIImage *image = (UIImage *)path;
                [image drawInRect:rect];
            }
            // 不是图片就画线
            else
            {
                [path.lineColor set];
                [path stroke];
            }
        }
    }
```

- 8、其他方法实现

```objc
    // 清屏
    - (void)eraseScreen
    {
        [self.drawLines removeAllObjects];
        [self setNeedsDisplay];
    }
    // 撤销
    - (void)undo
    {
        [self.drawLines removeLastObject];
        [self setNeedsDisplay];
    }
    // setter 方法，传入照片时进行刷新
    - (void)setImage:(UIImage *)image
    {
        _image = image;
        // 添加图片到绘图数组
        [self.drawLinesaddObject:_image];
        [selfsetNeedsDisplay];
    }
```

- 5.3、在控制器中实现

- 1、添加两个属性，imageView位于DrawView下面

![](../images/屏幕快照 2015-06-22 17.51.33.png)

```objc
    @property (nonatomic,strong) IBOutlet DrawView *drawView; // 绘图面板
    @property (weak, nonatomic) IBOutlet UIImageView *imageView; // 加载的图片显示面板
```

- 2、工具栏:通过拖线的方式创建一个方法

```objc
    // 清屏
    - (IBAction)eraseScreen:(UIBarButtonItem *)sender {
        [_drawView eraseScreen];
    }
    // 撤销
    - (IBAction)undo:(id)sender {
        [_drawView undo];
    }
    // 橡皮擦
    - (IBAction)eraser:(UIBarButtonItem *)sender {
        if ([sender.title isEqualToString:@"橡皮擦"]) {
            sender.title  = @"绘图";
            _drawView.lineColor = [UIColor whiteColor];
            _drawView.lineWidth = 5;
        }
        else if ([sender.title isEqualToString:@"绘图"]) {
            sender.title  = @"橡皮擦";
            _drawView.lineColor = [UIColor blackColor];
            _drawView.lineWidth = 1;
        }
    }


    // 保存图片到相册
    - (IBAction)save:(id)sender {
         // 开启上下文
        UIGraphicsBeginImageContextWithOptions(self.drawView.bounds.size, NO, 0);
        // 获取上下文
        CGContextRef ref = UIGraphicsGetCurrentContext();
        // 渲染控制器到上下文
        [self.drawView.layerrenderInContext:ref];
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        // 关闭上下文
        UIGraphicsEndImageContext();


        // 保存画板的内容放入相册
        // image:写入的图片
        // completionTarget图片保存监听者
        // 注意：以后写入相册方法中，想要监听图片有没有保存完成，保存完成的方法不能随意乱写,必须传入这个方法didFinishSavingWithError
        UIImageWriteToSavedPhotosAlbum(image, self, @selector(image:didFinishSavingWithError:contextInfo:), nil);
        // 保存到桌面
    //    NSData *data = UIImagePNGRepresentation(image);
    //    [data writeToFile:@"/Users/song/Desktop/sss.png" atomically:YES];
    }
    // 监听保存完成，必须实现这个方法
    - (void)image: (UIImage *) image didFinishSavingWithError: (NSError *) error contextInfo: (void *) contextInfo
    {
        NSLog(@"图片保存成功");
    }
```
- 3、加载图片：图片控制器

```objc
    // 从相册加载一张图片
    - (IBAction)photo:(id)sender {
        // 初始化图片控制器
        UIImagePickerController *pick = [[UIImagePickerController alloc] init];
        // 设置路径
        // 设置选择控制器的来源
        // UIImagePickerControllerSourceTypePhotoLibrary 相册集
        // UIImagePickerControllerSourceTypeSavedPhotosAlbum:照片库
        pick.sourceType = UIImagePickerControllerSourceTypeSavedPhotosAlbum;
        // 设置代理
        pick.delegate = self;
        // modal 方式弹出图片控制器
        [selfpresentViewController:pick animated:YEScompletion:nil];
    }
    // 图片代理
    - (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
    {
        // 获取选中的图片
        UIImage *image = info[@"UIImagePickerControllerOriginalImage"];
        NSLog(@"%@",image);
        self.imageView.alpha = 1; // 可见
        self.imageView.userInteractionEnabled = YES; // 设置交互
        self.imageView.image = image;
        [self addGestures]; // 添加手势
        // 退出控制器
        [self dismissViewControllerAnimated:YES completion: nil];
    }
```
- 4、添加各种手势

```objc
    - (void)addGestures
    {
        // 滑动
        UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(pan:)];
         pan.delegate = self;
        [self.imageViewaddGestureRecognizer:pan];
        // 旋转
        UIRotationGestureRecognizer *rotate = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotate:)];
        rotate.delegate = self;
        [self.imageViewaddGestureRecognizer:rotate];
        // 捏合
        UIPinchGestureRecognizer *pinch = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinch:)];
        pinch.delegate = self;
        [self.imageViewaddGestureRecognizer:pinch];
        // 长按
        UILongPressGestureRecognizer *lPress = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(lPress:)];
        [self.imageViewaddGestureRecognizer:lPress];
        // 点按
        UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tap:)];
        [self.imageViewaddGestureRecognizer:tap];
    }
    // 同一时刻运行响应多个手势
    - (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
    {
        return YES;
    }
    - (void)pan:(UIPanGestureRecognizer *)pan
    {
        CGPoint curP = [pan translationInView:self.imageView];
        self.imageView.transform = CGAffineTransformTranslate(self.imageView.transform, curP.x, curP.y);
        // 复位
        [pan setTranslation:CGPointZeroinView:self.imageView];
    }
    - (void)pinch:(UIPinchGestureRecognizer *)pinch
    {
        CGFloat scale = pinch.scale;
        self.imageView.transform = CGAffineTransformScale(self.imageView.transform, scale, scale);
        // 复位
        pinch.scale = 1;
    }
    - (void)tap:(UITapGestureRecognizer *)tap
    {
    }
    - (void)lPress:(UILongPressGestureRecognizer *)lPress
    {
        // 截取图片到绘图面板
        if (lPress.state == UIGestureRecognizerStateBegan)  {
             [UIViewanimateWithDuration:0.25animations:^{
                 self.imageView.alpha = 0.5;
             } completion:^(BOOL finished) {
                [UIViewanimateWithDuration:0.25animations:^{
                    self.imageView.alpha = 1;
                } completion:^(BOOL finished) {
                    // 截屏,使用之前定义的分类，包含头文件即可使用
                    UIImage *image = [UIImage imageWithSnipView:self.drawView];
                    self.imageView.alpha = 0;
                    _drawView.image = image; // 显示图片到绘图面板
                }];
             }];
        }
    }
    - (void)rotate:(UIRotationGestureRecognizer *)rotate
    {
        CGFloat rotation = rotate.rotation;
        self.imageView.transform = CGAffineTransformRotate(self.imageView.transform, rotation);
        // 复位
        rotate.rotation = 0;
    }
```
- 5、线条颜色和宽度设置

```objc
    - (IBAction)colorChange:(UIButton *)sender {
        _drawView.lineColor = sender.backgroundColor;
    }
    - (IBAction)valueChange:(UISlider *)sender {
        _drawView.lineWidth = sender.value * 10;
    }
```

- 效果

![](../images/画板.gif)




http://www.cnblogs.com/songliquan/

