# 项目总结

## tabBar默认显示蓝色的文字和图片
- 如果想显示自己的图片和文字，可以指定图片的渲染方式为原始方式
`image = [image imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal]`
- 还可以在`../images.xcassets`中设置图片的渲染方式为原始。

## appearance属性
- 如果想统一设置tabBar的某些属性，可以使用appearance属性来设置
    - 通过appearance统一设置所有UITabBarItem的文字属性
    - 后面带有`UI_APPEARANCE_SELECTOR`的方法, 都可以通过appearance对象来统一设置`

```objc
    UITabBarItem *item = [UITabBarItem appearance];

    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    dict[NSFontAttributeName] = [UIFont systemFontOfSize:12];
    dict[NSForegroundColorAttributeName] = [UIColor grayColor];
    [item setTitleTextAttributes:dict forState:UIControlStateNormal];

    NSMutableDictionary *dictSelected = [NSMutableDictionary dictionary];
    dictSelected[NSFontAttributeName] = [UIFont systemFontOfSize:12];
    dictSelected[NSForegroundColorAttributeName] = [UIColor darkGrayColor];
    [item setTitleTextAttributes:dictSelected forState:UIControlStateSelected];

```
## 自定义tabBar
- 系统自带的tabBar不能满足需求
- 自己定义UITabBar
    - 自定义一个类继承自UITabBar
    - 实现initWithFrame和layoutSubviews方法即可。

![](../images/屏幕快照 2015-07-22 21.00.27.png)

# 自定义UIView工具类
- 因为要经常获取控件的frame、height等属性进行设置，这里对UIView写一个分类，添加一些自定义的setter和getter方法
- 这样做得依据是
    - 分类中声明@property, 只会生成方法的声明, 不会生成方法的实现和带有_下划线的成员变量
    - 所以可以直接写几个关于这个常用属性的方法


## 设置所有的子控制器的返回按钮样式一致

![](../images/屏幕快照 2015-07-22 23.53.38.png)

```objc
//#import "SLQNavigationController.h"

@implementation SLQNavigationController

// 自定义导航控制器，在弹出窗口之前设置返回按钮样式。
// 每一个控制器弹出之前都会调用这个方法，所以可以保证所有的控制器的返回按钮样式一致。
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    // 判断是不是第一个控制器，如果不是就设置返回按钮
    if (self.childViewControllers.count > 0) {
        UIButton *backBtn = [UIButton buttonWithType:UIButtonTypeCustom];
        // 设置图片
        [backBtn setImage:[UIImage imageNamed:@"navigationButtonReturn"] forState:UIControlStateNormal];
        [backBtn setImage:[UIImage imageNamed:@"navigationButtonReturnClick"] forState:UIControlStateHighlighted];
        [backBtn setTitle:@"返回" forState:UIControlStateNormal];
        // 让按钮内部的所有内容左对齐
        backBtn.contentHorizontalAlignment = UIControlContentHorizontalAlignmentLeft;

        backBtn.size = CGSizeMake(70, 30);
        // 文字颜色
        [backBtn setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
        [backBtn setTitleColor:[UIColor redColor] forState:UIControlStateHighlighted];
        // 设置内边距，使得按钮紧靠边
        [backBtn setContentEdgeInsets:UIEdgeInsetsMake(0, -15, 0, 0)];

        [backBtn addTarget:self action:@selector(back) forControlEvents:UIControlEventTouchUpInside];

        viewController.navigationItem.leftBarButtonItems = @[
                                                             [[UIBarButtonItem alloc] initWithCustomView:backBtn]
                                                             ];
        // 隐藏tabBar
        viewController.hidesBottomBarWhenPushed = YES;

    }
    // 设置过控制器后再弹出
    // 这句super的push要放在后面, 让viewController可以覆盖上面设置的leftBarButtonItem
    [super pushViewController:viewController animated:animated];
}
- (void)back
{
    [self popViewControllerAnimated:YES];
}
@end
```

## 自定义cell
- cell被选中时其内部子控件会进入高亮状态，但是当将cell的属性selecte设置为none时，就不会显示高亮状态
I- 左边cell保存右边cell的用户数据，解决重复发生请求问题
- 添加cell左右间隔
	- 添加一个空白的UIView
	- 1、将所有子控件添加到一个UIView中，并且设置大小小于cell
	- 2、调整cell的contentView
	- 3、直接调整cell的frame
		- 直接修改不起作用，重新setFrame方法
		- 修改x值，width和height
		- 重写setFrame和setBounds方法可以保证自己的控件始终大小如一，别人无法修改。
- 表格刷新
	- header
		- 首次进入时刷新一次
		- 然后每次下拉刷新时刷新一次
			- 刷新之前清空user数组

	- footer
		- 每次选中左边cell时就判断footer是否显示
		- 主要是根据用户数量和服务器返回的总数比较得出
		- 如果加载完毕就显示noMoreData
		- 否则就停止刷新状态，等待用户再次刷新
- 问题
	- 1、重复下载数据 解决方法：判断用户数量是否大于0
	- 2、网络延迟带来的细节问题：点击左边某个cell后，右边tableView会显示残留数据
	- 解决方法:如果要重新下载数据，那么先刷新表格，显示当前cell对应的最新用户数据
	- 3、关于加载第二页数据的问题
	-  解决方法：记录当前的页码，根据页码加载数据，页码的大小不是关键，关键在于footer的状态
	- 4、重复发送请求
	-  解决方法：通过记录最新的参数列表来判断是不是最新的请求，如果不是就把数据保存起来，以备下次使用，是的话就刷新表格了

## 键盘处理
- inputView
- 设置文本输入框的属性`return key` :
- 监听键盘中`右下角按键`的点击，设置代理：`shouldReturn`
- 自定义键盘
	- 修改属性`inputView`
	- 工具条(键盘顶部)：`inputAccessoryView`
		- `UIToolBar`
- File's Owner
	- Owner : 监听对象（xib中class要和监听对象一致）
	- 加载xib指定owner
	- 指定Owner的话，耦合性太强，自定义UIToolBar解耦
- 自定义`UIToolBar`
	- 设置代理或者通知来处理点击事件

## 问题
- 1、忘记设置数据源
	- tableView一直不显示数据
## 登陆注册

### 设置状态栏

```objc
typedef NS_ENUM(NSInteger, UIStatusBarStyle) {
    UIStatusBarStyleDefault                                     = 0, // Dark content, for use on light backgrounds
    UIStatusBarStyleLightContent     NS_ENUM_AVAILABLE_IOS(7_0) = 1, // Light content, for use on dark backgrounds
};

/**
 *	设置状态栏显示状态
 */
- (UIStatusBarStyle)preferredStatusBarStyle{
    return UIStatusBarStyleLightContent;
}
```
### 自定义按钮
- 这个改变任意，只需在layoutSubviews里改变frame即可

### 登陆界面
- 显示按钮背景图片圆角
	- 代码
	```objc
	  // 显示圆角
    self.view.layer.cornerRadius = 5;
    self.view.layer.masksToBounds = YES;
	```
- XIB中kvc设置

### 占位文字颜色设置
	- UIColor(不行)
	- attribute(可以)
- 富文本显示
	```objc
	    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    dict[NSForegroundColorAttributeName] = [UIColor redColor];
    NSMutableAttributedString *attr = [[NSMutableAttributedString alloc] initWithString:@"手机号" attributes:dict];
    [attr addAttributes:@{NSForegroundColorAttributeName : [UIColor whiteColor] } range:NSMakeRange(0, 1)];
    [attr addAttributes:@{NSForegroundColorAttributeName : [UIColor redColor],
                          NSFontAttributeName : [UIFont systemFontOfSize:30]
                          } range:NSMakeRange(1, 1)];
    [attr addAttributes:@{NSForegroundColorAttributeName : [UIColor greenColor] } range:NSMakeRange(1, 1)];
    self.phoneField.attributedPlaceholder = attr;
	```
- 运行时时获取

```objc
	//#import <objc/runtime.h>
	 unsigned int count = 0;
    // 获取所有隐藏属性
    Ivar *ivas = class_copyIvarList([UITextField class], &count);
    for(int i = 0 ; i < count ; i ++)
    {
        Ivar iv = *(ivas + i);
        // _placeholderLabel
        SLQLog(@"%s",ivar_getName(iv));
    }
    // 释放
    free(ivas);
```

- 如果想在进入输入状态时显示不同的文字状态

```objc
- (void)awakeFromNib
{
//    [self setValue:[UIColor whiteColor] forKeyPath:@"_placeholderLabel.textColor"];
    // 设置占位符颜色
    self.tintColor = self.textColor;
    [self resignFirstResponder];
}
/**
 *	进入编辑状态，显示白色文字背景
 */
- (BOOL)becomeFirstResponder
{
    [self setValue:self.textColor forKeyPath:@"_placeholderLabel.textColor"];
    return [super becomeFirstResponder];
}
/**
 *	退出编辑状态，显示默认背景
 */
- (BOOL)resignFirstResponder
{
    [self setValue:[UIColor grayColor] forKeyPath:@"_placeholderLabel.textColor"];
    return [super resignFirstResponder];
}
```
### 更新推送通知

- 根据版本决定是否显示提示界面

```objc
/**
 *	关闭推送通知
 */
- (IBAction)closePushView:(id)sender {
    [self removeFromSuperview];
}
/**
 *	是否显示窗口
 */
+ (void)show
{
    // 判断是否是最新版本
    // 获取当前软件的版本号
    NSString *currentVersion = [NSBundle mainBundle].infoDictionary[@"CFBundleShortVersionString"];
    // 获取沙盒中存储的版本号
    NSString *sanboxVersion = [[NSUserDefaults standardUserDefaults] stringForKey:@"CFBundleShortVersionString"];
    if (![sanboxVersion isEqualToString:currentVersion]) {
         // 显示通知界面
        UIWindow *window = [[UIApplication sharedApplication] keyWindow];
        SLQPushGuideView *pushView = [SLQPushGuideView pushGuideView];
        pushView.frame = window.bounds;
        [window addSubview:pushView];

        // 保存版本号
        [[NSUserDefaults standardUserDefaults] setObject:currentVersion forKey:@"CFBundleShortVersionString"];
        // 立即写入
        [[NSUserDefaults standardUserDefaults ] synchronize];
    }
}
/**
 *	返回对象
 */
+ (instancetype)pushGuideView
{
    return  [[[NSBundle mainBundle] loadNibNamed:NSStringFromClass(self) owner:nil options:nil] lastObject];
}
```

-


## 精华模块
- 使用一个cell，显示不同界面的信息（4个）)


- 段子
	- 中间只显示一串文字
	- 上拉刷新和下拉刷新
	- 刷新成功与失败注意页码状态的切换
	- 比较时间差值 ： NSCanlader

```objc
@implementation NSDate (SLQDate)
- (NSDateComponents *)deltaFrom:(NSDate *)from
{
    NSCalendar *cal = [NSCalendar currentCalendar];

    NSCalendarUnit unit = NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay | NSCalendarUnitHour | NSCalendarUnitMinute | NSCalendarUnitSecond;
   return [cal components:unit fromDate:from  toDate:self options:0];
}
@end
```
### 时间显示

```objc
/** 显示方式
 今年
    今天
        1分钟内
            刚刚
        1小时内
            xx分钟前
        其他
            xx小时前
    昨天
        昨天 18:56:34
    其他
        06-23 19:56:23

 非今年
    2014-05-08 18:45:30
 */
```
- 头像右下角：微博认证（小图片）
- 重构：搞一个父类控制器处理相同类型的数据

## 精华模块
- 计算cell高度
	- cell frame
- 图片

![](../images/屏幕快照 2015-07-29 11.30.36.png)

- 图片下载进度
	- 各种细节

- 删除第三方框架时问题一大堆
	- 解决方式：使用继承。更换时只需该改变父类即可
	- 把通用的方法写出来，其他的功能全部封装到自己额类中


- 长图显示最顶部
	- 使用2d绘图方式获取，等比例缩放


### 加号按钮动画效果

- pop facebook（弹簧效果）

- pop和Core Animation的区别
	- 1.Core Animation的动画只能添加到layer上
	- 2.pop的动画能添加到任何对象
	- 3.pop的底层并非基于Core Animation, 是基于CADisplayLink
	- 4.Core Animation的动画仅仅是表象, 并不会真正修改对象的frame\size等值
	- 5.pop的动画实时修改对象的属性, 真正地修改了对象的属性


![](../images/屏幕快照 2015-07-30 09.17.26.png)

- 弹出控制器

- 弹出UIView

- 自定义UIWindow

- 自定义状态栏显示通知



	- 版本号
	![](../images/屏幕快照 2015-07-30 14.45.59.png)
- 框架细则
	- 把项目文件和资源文件放到和项目同名的文件夹中
	- 把图片资源打包到bundle中，直接把文件夹后缀名改为.bundle即可，访问图片时添加bundle名称
- 上传到github
- 上传到CocoaPod
	- pod trunk

## UIDynamic 物理引擎


### 声音界面
### 视频界面
- 这两个界面实现和图片的差不多，只不过还是增加几个属性，然后设置以下具体的值

### 评论功能
2015-8-2 上午

- cell底部最热评论

- 评论具体信息
	- 键盘通知
- header footer
	- UITableViewHeaderFooterView
	- 和cell类似，有缓存池，contentViwe,可以注册

```objc
//#import "SLQHeaderFooterView.h"

@interface SLQHeaderFooterView ()
/**lable*/
@property (nonatomic, weak) UILabel *lable;
@end
@implementation SLQHeaderFooterView
static NSString *ID = @"header";
+ (instancetype)headerFooterView:(UITableView *)tableView
{
    SLQHeaderFooterView *header  = [tableView dequeueReusableHeaderFooterViewWithIdentifier:ID];
    if (header == nil) {
        header = [[SLQHeaderFooterView alloc] initWithReuseIdentifier:ID];
    }
    return header;
}

- (instancetype)initWithReuseIdentifier:(NSString *)reuseIdentifier
{
    if (self = [super initWithReuseIdentifier:reuseIdentifier]) {

        self.contentView.backgroundColor = SLQAppBG;

        UILabel *lable  = [[UILabel alloc] init];
        lable.textColor = SLQRGBColor(67, 67, 67);
        lable.x = SLQCellMargin;
        lable.width = SLQScreenW - 2* lable.x;
        lable.autoresizingMask = UIViewAutoresizingFlexibleHeight;
        [self.contentView addSubview:lable];
        self.lable = lable;
    }
    return self;
}
- (void)setTitle:(NSString *)title
{
    _title = [title copy];
    self.lable.text = title;
}
@end
```

- 模型和字典映射

```objc
// 模型和属性映射
+ (NSDictionary *)replacedKeyFromPropertyName
{
    return @{
             @"small_image":@"image0",
             @"middle_image":@"image1",
             @"large_image":@"image2",
             @"ID":@"id",
             @"top_cmt":@"top_cmt[0]"// 数组中具体元素
             };
}
```

- cell auto-sizing（iOS8）

- cell 高度自动计算（控件固定）
	- 让cell bottom等于最底部控件的bottom
	- `关键是contentView的bottom等于最下面的控件的bottom`

```objc
   // 自定cell的估计高度并执行自动计算高度
    self.tableView.estimatedRowHeight = 44;
    self.tableView.rowHeight = UITableViewAutomaticDimension;
```
- 上拉刷新
- 重复请求:保存参数、使用manage

```objc
	  // 加载数据之前，取消所有请求
    [self.manager.tasks makeObjectsPerformSelector:@selector(cancel)];
```

- 点击状态栏回到scrollView顶部
	- UIWindow
	- 坐标系转换 convertRect: toView:
	- 判断当前显示界面是不是当前window

```objc
/**
 *	是否在主窗口上
 */
- (BOOL)isShowingOnKeyWindow
{
    // 坐标转换
    CGRect newRect =  [self.superview convertRect:self.frame toView:nil];
    CGRect keyBounds = SLQKeyWindow.bounds;
    // 判断两个控件是否有重叠
    BOOL isOverlaped = CGRectIntersectsRect(newRect,keyBounds);
    return (self.window == SLQKeyWindow) && !self.hidden && self.alpha > 0.01 && isOverlaped;
}
```

### 2015-8-3

- 添加过自定义winow后，登录界面状态栏显示有问题：不是白色
	- 方法1：改变状态栏控制器方式：关闭控制器管理状态栏，直接使用UIApplication 管理状态栏(info.list里设置)
	![](../images/屏幕快照 2015-08-05 23.39.01.png)

	- 方法2：在登录界面隐藏window，退出登录时再显示
```objc
    // 设置状态栏为默认
  [SLQTopWindow show];
//[UIApplication sharedApplication].statusBarStyle = UIStatusBarStyleDefault;
```
- 给github其他项目提交代码
	- 1、fork到自己仓库中
	- 2、下载到本地，进行修改
	- 3、同步到自己服务器
	- 4、pull request
	- 5、作者审核并 merge

#### menuController
- 长按某个控件，弹出一个小菜单
	- 默认支持menuController的控件
		- UIWebView
		- UITextField
		- UITextView
	- 其他控件需要实现可以使用自定义
		- canPerform
	- 功能选择，使用
	- 自定义
#### 圆形头像
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

### 第二次选中tabBar时刷新控制器

- 1、在AppDelegate选中时发送通知，在各个控制器中接受通知

```objc
// 实现代理方法：监听tabBar点击
- (void)tabBarController:(UITabBarController *)tabBarController didSelectViewController:(UIViewController /*)viewController
{
    // 点击过后发送通知
    [[NSNotificationCenter defaultCenter] postNotificationName:SLQNotificationTabBarClickedKey object:nil userInfo:nil];
}

//---------------
// 在控制器中接收事件并处理
// 接收tabBar选中通知
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(tabBarClick) name:SLQNotificationTabBarClickedKey object:nil];

/**
 *	选中tabBar就调用
 */
- (void)tabBarClick
{
    BOOL isTopicView = self.tabBarController.selectedViewController == self.navigationController ; // 是不是
    BOOL isShowing = [self.view isShowingOnKeyWindow] ; // 是不是正在显示的控制器
    BOOL isClickTwice = (self.clickTabBarCount == self.tabBarController.selectedIndex) ; // 记录点击的控制器索引

     if(isTopicView && isShowing && isClickTwice) // 是精华控制器并且点击两次,就刷新数据
    {
        SLQLog(@"%zd",self.tabBarController.selectedIndex);

        [self.tableView.header beginRefreshing];
    }
    // 记录选中控制器的索引
    self.clickTabBarCount = self.tabBarController.selectedIndex;
}
```

- 2、监听tabBar按钮的点击addTarget，发送通知
		- 重复添加
		- 解决方法：声明一个bool变量进行控制
```objc

// 静态局部变量记录按钮添加次数，只添加一轮
static BOOL addTar = NO;”””

if(!addTar)
{
    [btn addTarget:self action:@selector(btnClick:) forControlEvents:UIControlEventTouchUpInside];
}
```

### 新帖模块
- 直接继承实现，改变请求参数

```objc
/**
 *	请求参数a
 */
- (NSString *)a
{
    if ([self.parentViewController isKindOfClass:[SLQNewViewController class]]) {
         return @"newlist";
    }
    return @"list";
}
```
- 评论模块中，评论为0时服务器传回空数组，进行判断.

```objc
// 如果返回数据为nil
if (![responseObject isKindOfClass:[NSDictionary class]]) {
     [self.tableView.header endRefreshing];
    return ;
}
```
### 我的模块
- tableView:group
- 添加按钮
- 计算行数
*公式：总页数 = （总个数 + 列数 - 1）/ 列数*

```objc
    // 计算按钮总高度
//    NSInteger height = self.squares.count / cols;
//    CGFloat res = self.squares.count % cols;
//    if(res != 0)
//    {
//        height ++;
//    }
//
////    self.height = height;
    // 公式：总页数 = （总个数 + 列数 - 1）/ 列数
    self.height =  btnH * (self.squares.count + cols - 1) / cols ;
```
#### 网页加载进度条

- 框架 `NJKWebViewProgress`

```objc

    self.progress = [[NJKWebViewProgress alloc] init];
    self.webView.delegate = self.progress;
    __weak typeof(self) weakSelf = self;
    self.progress.progressBlock = ^(float progress)
    {
        [weakSelf.progressView setProgress:progress animated:YES];
        weakSelf.progressView.hidden = progress == 1.0;
    };
    // 如果想在控制器继续实现代理方法，那么需要再将代理设置回来
    self.progress.webViewProxyDelegate = self;
```

## 2015-8-5
- 添加段子
	- 给每个段子添加一个标签，用于快速索引`tag`
	- 导航栏中间title文字属性 `setTextAttribute`
	- 可以通过`apprance`统一设置
	- `apperence`设置的属性可能会有延迟，在viewDidLoad里可以强制刷新
	`[self.navigationController.navigationBar layoutIfNeeded];`

### 自定义UITextField（增加占位文字功能）

- 创建带有占位文字功能的textField

- 文字编辑
	- `UITextField`
		- 只能显示一行，有占位文字
	- `UITextView`
		- 可以显示多行文字，没有占位文字
- 实际需要：可以显示多行，又有占位文字
- 方法：继承UITextView，然后再增加占位文字功能
	- placeholder 实现
- drawRect实现占位文字
	- 无法随光标滚动而滚动

- presentedViewController --> 指向弹出的控制器
- presentngViewController --> 指向弹出控制器的父控制器

- textField
	- 文字改变不建议使用代理方法（使用代码改变文字不会进入代理方法）
	- 通知
	- addtarget
	`// 监听textField的状态改变`
   ` [textField addTarget:self action:@selector(textFieldChanged) forControlEvents:UIControlEventEditingChanged];`
	- 代理实现returnKey的监听

```bojc
/**
 *	监听键盘最右下角按钮的点击（return key， 比如“换行”、“完成”等等）
 */
- (BOOL)textFieldShouldReturn:(UITextField *)textField
{
    SLQLogFunction;
    if (textField.hasText) {
        [self addTag];
    }
    return YES;
}
```

- 协议方法：insertText： 特殊字符以及英文字符，不能相应汉字输入


- 协议方法：deleteBackword： ： 删除键
	- block实现

```objc
/**
 *	删除键block
 */
@property (nonatomic, copy) void (^deleteBlock)();
```

## 2015-8-6
### 上午(视频再看一遍)
- 子控件的布局，最好放到viewDidLayoutSubviews
- 1、获取缓存大小
	- sd_webimage   [SDImageCache sharedImageCache].getsize;
- 2、使用NSFileMangler
	- 遍历器对文件夹进行遍历，然后累加文件大小
	- `NSDirectoryEnumerator`
	- 默认只会遍历文件，所以要把子文件夹跳过去 `fileExistsAtPath：`
	- 通过attrbute 的属性：文件类型来判断`NSFileType`

	- `contentDirectoryAtPath:()`
	- `subPathAtPath:`
- 清空缓存
- 方式1：使用SDWebImage自带的方法
```objc
    // 获取缓存大小
    NSUInteger size = [SDImageCache sharedImageCache].getSize;
    // 清空缓存
//    [[SDImageCache sharedImageCache] clearDisk];
```
- 方式2:
- 文件遍历器，

```objc
NSFileOwnerAccountID : 501,
NSFileSystemFileNumber : 10163664,
NSFileExtensionHidden : 0, //
NSFileSystemNumber : 16777220,
NSFileSize : 16594, // 大小
NSFileGroupOwnerAccountID : 20,
NSFilePosixPermissions : 420,
NSFileCreationDate : 2015-08-12 00:39:36 +0000, // 创建时间
NSFileType : NSFileTypeRegular, // 文件类型：文件/文件夹
NSFileGroupOwnerAccountName : staff,
NSFileReferenceCount : 1,
NSFileModificationDate : 2015-08-12 00:39:36 +0000
```

```objc
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *cachePath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
   // "default/com.hackemist.SDWebImageCache.default"
    NSString *cacheFullPath = [cachePath stringByAppendingPathComponent:@"default/com.hackemist.SDWebImageCache.default"];
    NSDirectoryEnumerator *fileEnumerator = [fileManager enumeratorAtPath:cacheFullPath];
    NSUInteger cacheSize = 0;
    for (NSString *fileName in fileEnumerator) {
        NSString *filePath = [cacheFullPath stringByAppendingPathComponent:fileName];
        // 使用文件属性
        NSDictionary *attrs = [[NSFileManager defaultManager] attributesOfItemAtPath:filePath error:nil];
        // 获取每个文件的文件大小NSFileSize
        cacheSize += [attrs[NSFileSize] integerValue];
    }
    NSLog(@"%zd",cacheSize);
```
- 如果路径中还包含文件夹，那么就得使用另一种方式
- 可以直接递归读取文件夹，累加文件

```objc
   // 使用文件属性
        NSDictionary *attrs = [[NSFileManager defaultManager] attributesOfItemAtPath:filePath error:nil];
        BOOL isDir = NO;
        [fileManager fileExistsAtPath:filePath isDirectory:&isDir];
        if (isDir) {// 是文件夹，跳过
            continue;
        }
```
### app换肤功能
- 建立一个plist文件，里面保存各种颜色，通过key来改变颜色

- 通过文件夹方式：每中皮肤放在一个文件夹下，并且名称一致
- 然后换肤的话通过只需修改路径即可
-
### 下午（视频看一遍）
- 动态相册
	- UIScrollView
	- UICollectionView 瀑布流布局
		- 水




## 2015-8-7
### 上午
- 瀑布流布局
	- UICollectionView
	- 自己排布cell
- 运行时
	- 属性和成员变量
	- Method Sizzle
	- 交换方法执行
	- 字典、数组value为nil










