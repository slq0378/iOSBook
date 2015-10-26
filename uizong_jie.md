# UI总结

## 1、程序启动后的开始动画

- 程序启动后可以加载一个简单的动画界面来介绍程序或者用户信息。

- 可以使用一个xib来描述界面。并且如果想在程序加载完成后第一个加载这个xib文件，需要在Appdelegate中手动加载这个xib

```objc
// 通过stroyboard启动，跟控制器的view并不会在程序启动完成的时候添加到窗口，属于懒加载范畴
// 程序启动完成的时候调用
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    // 创建窗口
    self.window = [[UIWindowalloc] initWithFrame:[UIScreenmainScreen].bounds];

    // 加载storyboard
    UIStoryboard *story = [UIStoryboardstoryboardWithName:@"Main"bundle:nil];

    UIViewController *vc = [story instantiateInitialViewController];

    self.window.rootViewController = vc;

    // 显示窗口
    [self.windowmakeKeyAndVisible];

    // 加载欢迎界面
    WelcomView *welcome = [WelcomView welcomView];
    // 设置位置和尺寸
    welcome.frame = self.window.bounds;
    // 插入到最外层
    [self.window addSubview:welcome];

    returnYES;
}
```
- 在xib中如果想给内部子控件添加动画，并且动画和主界面的出现有延迟,需要在以下某个方法中实现。

```objc
// 视图添加到窗口时会调用方法didMoveToWindow 再调用didMoveToSuperview
- (void)didMoveToWindow
{
    NSLog(@"%s",__func__);

}
// 视图添加到窗口时会调用这个方法
// 将后续控件的动画写道这个方法中

- (void)didMoveToSuperview
{
    NSLog(@"%s",__func__);

}
```

## 2、LaunchScreen.xib

- 这个系统自动生成的xib文件在应用启动时会自动加载，可以在里面添加一些控件，但是注意：LaunchScreen比LaunchImage优先级高。
- 设置LaunchImage需要注意，默认模拟器的尺寸跟启动图片有关系。

```objc
// 加载类的时候调用
// 当程序一启动的时候就会调用
+ (void)load
{
    NSLog(@"%s",__func__);
}
// 当前类或者他的子类第一次使用的时候才会调用
+ (void)initialize
{
    NSLog(@"%s",__func__);
}
```

## 3、给插件添加自定义的提醒方式

- 自定义的分类，想让这个方法弹出加载图片提醒

```objc
#import "UIImage+SLQImage.h"

@implementation UIImage (SLQImage)
// 按照原始图片渲染照片,默认导航条渲染位蓝色
+ (instancetype)imageWithOriImage:(NSString *)imageName
{
    UIImage *image = [UIImage imageNamed:imageName];
    return [image imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
}
@end
```
- 把这个方法添加到插件中再次编译运行就可。

- 打开插件工程，找到配置文件，是一个plist文件，然后添加需要的条目。

## 4、给分类添加属性的话，只会生成set和get方法

```objc
#import <UIKit/UIKit.h>

@interface UIView (SLQFrame)
// 分类里面不能生成成员属性
// 会自动生成get，set方法和成员属性

// @property如果在分类里面只会生成get,set方法的声明，并不会生成成员属性。
@property (nonatomic, assign) CGFloat height;
@property (nonatomic, assign) CGFloat width;
@property (nonatomic, assign) CGFloat x;
@property (nonatomic, assign) CGFloat y;

@end
```

## 5、如何抛出异常

```objc
 NSInteger count = items.count; // 按钮个数

// 判断列数是不是3的倍数，如果不是就抛出异常

if(count % 3)
{
   // 跑出异常
    NSException *excep = [NSException exceptionWithName:@"列数有误" reason:@"列数不是3的倍数" userInfo:nil];
    [excep raise];// 抛出异常
}
// 运行结果
//  *** Terminating app due to uncaught exception '列数有误', reason: '列数不是3的倍数'
```

## 6、自定义tabBar

- 默认barBar显示的图片高度不能超过44,如果大于44显示出问题，这是可以自定义tabBar。

- 6.1、新建一个类SLQTabBar,因为tabBar的个数不确定，所以需要外界传入，这里使用模型数组来设置。

```objc
//  模仿下UITabBar
// UITabBar里面的按钮由UITabBarController的子控制器
@interface SLQTabBar : UIView
// 模型数组(UITabBarItem)，直接使用系统的模型即可
@property (nonatomic, strong) NSArray *items;
@end
```
- 6.2、模型数组只需在初始化时添加一次即可，所以使用懒加载方式.

```objc
@interfaceSLQTabBar ()
@property (nonatomic, weak) SLQTabBarButton *selected; // 按钮是否选中

@end

- (void)setItems:(NSArray *)items
{
    _items = items;

    // UITabBarItem保存按钮上的图片,按次序添加
    for (UITabBarItem * bar in items) {

        // 使用自定义的button，取消高亮状态
        UIButton *btn = [SLQTabBarButtonbuttonWithType:UIButtonTypeCustom];
        // 设置标签
        btn.tag = self.subviews.count;
        // 设置背景
        [btn setBackgroundImage:bar.imageforState:UIControlStateNormal];
        [btn setBackgroundImage:bar.selectedImageforState:UIControlStateSelected];
        // 添加按钮按下响应事件
        [btn addTarget:selfaction:@selector(btnClick:) forControlEvents:UIControlEventTouchDown];

        [self addSubview:btn];

        if (self.subviews.count == 1) {
            // 默认选中第一个
            [self btnClick:btn];
        }
    }
}
```
- 6.3、tabBar上是一排按钮排列而成，不过没有高亮状态，只有Normal和selected状态。
    - 这个只需要重写 UIButton的一个方法即可。新建一个继承自UIButton的类，重写 setHighlighted 方法

```objc
#import "SLQTabBarButton.h"
@implementation SLQTabBarButton
// 取消按钮高亮状态
- (void)setHighlighted:(BOOL)highlighted
{}
@end
```
- 6.4、每次添加过按钮之后都要对按钮进行布局

```objc
- (void)layoutSubviews
{
    [superlayoutSubviews];

    int count = (int)self.subviews.count;
    // 按照屏幕宽度计算按钮宽度
    for (int i = 0 ;  i < count;  i ++) {
        CGFloat x = 0;
        CGFloat y = 0;
        CGFloat w = [UIScreenmainScreen].bounds.size.width / count;
        CGFloat h = self.bounds.size.height;

        SLQTabBarButton *btn = self.subviews[i];

        x = i * w;
        btn.frame = CGRectMake(x, y, w, h);
    }
}
```
- 6.5、响应按钮点击的话，使用代理传递数据

```objc
// 定义
@classSLQTabBar;
// 添加代理,响应按钮点击
@protocol SLQTabBarDelegate <NSObject>
@optional
- (void)tabBar:(SLQTabBar *)tabBar didClickButton:(NSInteger ) index;
@end
// 声明代理
@property (nonatomic, weak) id<SLQTabBarDelegate> delegate;

//  使用
- (void)btnClick:(UIButton *)btn
{
    // 取消选中上一个按钮
    _selected.selected = NO;
    // 选中当前按钮
    btn.selected  = YES;
    // 记录选中的按钮
    _selected = btn;

    // 切换控制器，添加一个代理,先判断是否有这个方法
    if ([self.delegate respondsToSelector:@selector(tabBar:didClickButton:)])
    {
        // 发送消息到代理，通过按钮tag判断是哪一个按钮
        [self.delegate tabBar:self didClickButton:btn.tag];
    }
}
```
- 6.6、具体使用方法，在控制器中设置tabBar
    - 添加自定义的tabBar到tabBar，然后将系统生成的按钮删除.

```objc
//添加自定义的tabBar到tabBar，然后将系统生成的按钮删除
- (void)setAllTabBar
{
    // 移除所有子控件
//    [self.tabBar removeFromSuperview];

    // 添加自定义tabBar
    SLQTabBar *bar = [[SLQTabBar alloc] init];
    // 传入模型数据
    bar.items = self.items;
    // 设置尺寸
    bar.frame = self.tabBar.bounds;
    // 设置代理，监听按钮点击
    bar.delegate = self;
    // 添加到tabBar,但是要把系统自带的按钮删除
    [self.tabBar addSubview:bar];
}
```

- 移除系统按钮

```objc
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];

    // 删除系统自带的tabBar
    for (UIView *childView in self.tabBar.subviews) {
        if (![childView isKindOfClass:[SLQTabBar class]]) {
            [childView removeFromSuperview];
        }
    }
}
```

- 监听按钮点击

```objc
- (void)tabBar:(SLQTabBar *)tabBar didClickButton:(NSInteger)index
{
//    NSLog(@"%s--%d",__func__,index);
    // 切换控制器
    self.selectedIndex = index;
}
```

## 7、手动添加pch文件都项目

pch文件中包含的东西在项目中每个文件中都可以使用。

![](../images/手动添加pch文件到项目.gif)

## 8、load和initialize

```objc
// 当程序一启动的时候就会调用
+ (void)load
{
}
// 当前类或者他的子类第一次使用的时候才会调用
+ (void)initialize
{}
```

## 9、 给任意弹出的窗口添加遮盖
- 这个也比较常见，比如说弹出登陆界面时，程序其他部分都是灰色背景，且不能点击。

- 可以给程序添加一个灰色的View，而且要显示在所有界面之前，那么可以使用

`[UIApplication sharedApplication].keyWindow；`

- 添加到这个窗口上的窗口默认都在最前面。

- 自定义一个继承自UIView的类，添加两个方法即可。

```objc
// 屏幕尺寸
#define SLQScreenBounds [UIScreen mainScreen].bounds
// 主窗口
#define SLQKeyWindow [UIApplication sharedApplication].keyWindow

// 显示遮盖
+ (void)show
{
    // 获取主窗口，添加一个view 到主窗口
    UIView *view = [[SLQCoverView alloc] initWithFrame:SLQScreenBounds];
    view.backgroundColor = [UIColorblackColor];
    view.alpha = 0.4;
    // 添加窗口到主窗口,显示在最上层
   [SLQKeyWindow addSubview:view];

}
// 隐藏遮盖
+ (void)hide
{
    // 取出子窗口移除,首先判断是不是SLQCoverView类型的view
    for (UIView *childView in SLQKeyWindow.subviews) {
        if ([childView isKindOfClass:self]) {
            [childView removeFromSuperview];
        }
    }
}
```

## 10、loadView、viewDidLoad和viewDidUnLoad

- 作用
    - loadView 苹果设计这个方法就是给我们自定义UIViewController的view用的
    - viewDidLoad 在创建过控制器的view后，向view中添加其他控件
    - viewDidUnLoad  系统发出内存警告时

- 关系
    - 1.第一次访问UIViewController的view时，view为nil，然后就会调用loadView方法创建view
    - 2.view创建完毕后会调用viewDidLoad方法进行界面元素的初始化
    - 3.当内存警告时，系统可能会释放UIViewController的view，将view赋值为nil，并且调用viewDidUnload方法
    - 4.当再次访问UIViewController的view时，view已经在3中被赋值为nil，所以又会调用loadView方法重新创建view
    - 5.view被重新创建完毕后，还是会调用viewDidLoad方法进行界面元素的初始化

- View的生命周期

![](../images/view的生命周期.jpg)

## 11、UINavigationBar

- 所生成的图片都是经过处理的view，为蓝色

![](../images/屏幕快照 2015-06-30 15.14.26.png)


- 原始图片

![](../images/屏幕快照 2015-06-30 15.14.40.png)

```objc
// 默认左右两遍的view都是蓝色的，所以要返回一个原始的图片
UIImage *image = [UIImageimageNamed:@"CS50_activity_image"] ;
// 设置图片的渲染方式为原始图片
image = [image imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];

self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc] initWithImage:image style:UIBarButtonItemStylePlain target:self action:@selector(activeBtnClick)];
```

## 12、九宫格布局`UICollectionViewController`

![](../images/九宫格分析.jpg)

- 创建控制器一定要指定默认的布局样式。

```objc
// 加载一个九宫格布局的控制器,必须指定布局样式
UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
vc = [[SLQGuideCollectionController alloc] initWithCollectionViewLayout:layout];
// 也可以重写控制器的init方法

- (instancetype)init
{
    UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
    // 初始化必须指定默认的布局样式
    return [super initWithCollectionViewLayout:layout];
}
```

- 在指定布局时可以设置各种属性

```objc
- (instancetype)init
{
    UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
    // cell大小
    layout.itemSize = SLQScreenBounds.size;
    // 水平间距
//    layout.minimumInteritemSpacing = 0;
    // 垂直间距
    layout.minimumLineSpacing = 0;
    // 每一组距离四面的距离
//    layout.sectionInset = UIEdgeInsetsMake(0, 0, 0, 0);
    // 滚动方向
    layout.scrollDirection = UICollectionViewScrollDirectionHorizontal;

    // 初始化必须指定默认的布局样式
    return [super initWithCollectionViewLayout:layout];
}
```
- 如果想显示控件，那么使用方法和UITableView非常相似。使用代理设置每个CollectionView

```objc
// 一共多少组
- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView
{
    return 1;

}
// 某组多少元素
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    return  4;
}
// cell 的内容
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    // 只能以这种方式创建cell，首先注册xib或者类，然后去缓存池中取，如果缓存池没有就自己创建一个新的cell
    SLQGuideCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:IDforIndexPath:indexPath];
    // 设置cell内容
    NSString *imageName = [NSString stringWithFormat:@"guide%ldBackground",indexPath.item + 1];
    cell.image = [UIImage imageNamed:imageName];
    return cell;
}
```
## 13、运行时 - runtime

- 首先包含头文件，然后使用里面的方法即可

```objc
// 运行时对象
#import <objc/runtime.h>

    // 遍历某个类里面所有属性 Ivar:表示成员属性
    // copyIvarList只能获取哪个类下面的属性，并不会越界（不会把它的父类的属性给遍历出来）
    // Class 获取哪个类的成员属性
    // count:告诉你当前类里面成员属性的总数
    //    unsigned int count = 0;
    //    // 返回成员属性的数组
    //
    int count = 0;
    Ivar *name =   class_copyIvarList([UIGestureRecognizer class], &count);
    for (int i = 0; i < count; i ++) {
        Ivar ivar = name[i];
        // 获取属性名
        NSString *proName = @(ivar_getName(ivar));
         NSLog(@"%@",proName);
    }

//<UIScreenEdgePanGestureRecognizer: 0x7fb6ba46f1c0; state = Possible; delaysTouchesBegan = YES; view = <UILayoutContainerView 0x7fb6ba7481b0>; target= <(action=handleNavigationTransition:, target=<_UINavigationInteractiveTransition 0x7fb6ba46d8e0>)>>
//UIScreenEdgePanGestureRecognizer:手势类型
//target - _UINavigationInteractiveTransition
//action - handleNavigationTransition
```
- 可以看到系统默认在滑动手势时作的一些事情，指定了手势的target和action，可以利用这些方法来自定义自己的手势。

## 14、自定义系统默认生成的手势

- 1、首先要找到这个手势相关联的属性和方法。因为是私有的，不能直接看到，所以只能通过运行时机制获取属性名
    - 先通过运行时机制的一些函数，观察属性名，然后确定在自己要得。

```objc
int count = 0;
Ivar *name =   class_copyIvarList([UIGestureRecognizer class], &count);
for (int i = 0; i < count; i ++) {
    Ivar ivar = name[i];
    // 获取属性名
    NSString *proName = @(ivar_getName(ivar));
     NSLog(@"%@",proName);
}
```

- 2、找出属性名并添加手势

```objc
// 找到属性名
NSArray *arr = [self.interactivePopGestureRecognizer valueForKey:@“_targets"];
// 获取属性
id objc = [arr firstObject]; //  -(action=handleNavigationTransition:, target=<_UINavigationInteractiveTransition 0x7f7f93f5adf0>)
// 获取方法target
id target = [objc valueForKey:@"_target"]; //
// 添加自定义手势,指定方法位系统内部私有的方法handleNavigationTransition:
UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self.interactivePopGestureRecognizer.delegate action:@selector(handleNavigationTransition:)];
pan.delegate = self;
[self.view addGestureRecognizer:pan];
```

- 3、实现代理方法-只要不是根控制器就使用手势

```objc
// UIGestureRecognizerDelegate 代理方法
// 是否开始手势，如果是不是根控制器就开始手势
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer
{
      // 判断下当前控制器是否是跟控制器
    return (self.topViewController != [self.viewControllersfirstObject]);
}
```

## 15、高斯模糊实现

- 这个方法是别人封装好的，直接拿来使用即可。

```objc

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    if (indexPath.section == 2 && indexPath.row == 0) {
         // 弹出窗口,指定frame
        SLQSettingBlurView *blur = [[SLQSettingBlurView alloc] initWithFrame:SLQScreenBounds];
        [SLQKeyWindow addSubview:blur];
        // 弹出提示窗口
        [MBProgressHUD showSuccess:@"没有最新版本"];
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            // 隐藏高斯窗口
            [blur removeFromSuperview];
        });
    }
}
```

## 16、block 循环利用

```objc
// block会把代码里所有强指针全部强引用
// 解决循环利用的问题

//    __weak XMGScoreViewController *weakSelf = self;

// typeof获取括号里面的类型
__weak typeof(self) weakSelf = self;

// 在iOS7之后只要在cell上添加textField都自动做了键盘处理
item.itemUpdate = ^(NSIndexPath *indexPath)
{

    // 获取当前选中的cell
    UITableViewCell *cell = [weakSelf.tableView cellForRowAtIndexPath:indexPath];

    // 弹出键盘
    UITextField *textField = [[UITextField alloc] init];

    [textField becomeFirstResponder];

    [cell addSubview:textField];

};
```
## 16、json文件

- 文件中保存的时字典数据

```objc
[
  {
    "title" : "如何领奖？",
    "html" : "help.html",
    "id" : "howtoprize"
  },
  {
    "title" : "如何充值？",
    "html" : "help.html",
    "id" : "howtorecharge"
  },
]
```
-  直接当做普通文件读取就行

```objc
- (NSMutableArray *)items
{
    if (_items == nil) {
        _items = [NSMutableArray array];
        NSString *fullPath = [[NSBundle mainBundle] pathForResource:@"help.json" ofType:nil];

        NSData *data = [NSData dataWithContentsOfFile:fullPath];
        // 从文件读取数据
        NSDictionary *allData = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingAllowFragments error:nil];
        for (NSDictionary *dict in allData) {
            SLQHtmlItem *item = [SLQHtmlItem itemWithDict:dict];
            [_items addObject:item];
        }
//        NSLog(@"%@",allData);

    }
//    NSLog(@"%@",_items);
    return _items;
}
```
-  关键一句话是使用 NSJSONSerialization 对文件进行解析

```objc
  NSDictionary *allData = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingAllowFragments error:nil];
```

- 文件中读取的关键字和系统保留字有冲突
    - 那么读取时就要进行修正，使用KVC进行字典读取要保证，模型中得属性和字典中元素数量和名称一致，不能有任何差别。
    - 现在的问题就是，文件中又一项数据的key是id，这个和系统的保留字id冲突，必须对id进行转换。所以声明属性是大写ID。

```objc
@property (strong, nonatomic) NSString *ID;
 然后是初始化方法，使用 KVC进行读取

+ (instancetype)itemWithDict:(NSDictionary *)dict
{
    SLQHtmlItem *item = [[self alloc] init];
    // KVC，会出问题，id 和 ID无法自动转换
    [item  setValuesForKeysWithDictionary:dict];
    // 使用通用方法更加方便，抽出分类
//    SLQHtmlItem *item  = [SLQHtmlItem objectWithDict:dict mapDict:@{@"ID":@"id"}];

    return item;
}
```
- 重写setvalue方法，在方法里进行转换，这个可以应对数量较少的键值对冲突，如果有十几个或者更多，使用起来就很麻烦，要写一堆if语句。

```objc
// 因json文件中得字典key 是id，与系统关键字冲突，手动判断再返回正确的值
- (void)setValue:(id)value forKey:(NSString *)key
{
    // 判断是不是id
    if ( [key isEqualToString:@"id"]) {

        [self setValue:value forKey:@"ID"];

    }
    else
    {
        [super setValue:value forKey:key];
    }
}
```
- 更加方便的方法是抽出一个分类，因对更加复杂的情况，如果有10个key和系统关键字重复，这个分类比较好使。

```objc
// 快速进行字典转模型
// mapDict:模型中的哪个属性名跟字典里面的key对应
+ (instancetype)objectWithDict:(NSDictionary *)dict mapDict:(NSDictionary *)mapDict
{
    id obj = [[self alloc] init];
    // 运行时遍历模型中得属性
    unsigned int count = 0;
    Ivar *ivar = class_copyIvarList(self, &count);
    for (int i = 0 ; i < count ; i ++) {
        // 取出
        Ivar var = ivar[i];
        // 取出模型
        NSString *name = @(ivar_getName(var));
        // 输入如下，有一个_,直接去掉就可得到属性名
        // 菜皮[61496:622121] _title
        // 菜皮[61496:622121] _html
        // 菜皮[61496:622121] _ID
        // 去除下划线
        name = [name substringFromIndex:1];

        id value = dict[name];

        // 需要由外界通知内部，模型中属性名对应字典里面的哪个key
        // ID -> id
        if (value == nil) {
            //
            if(mapDict)
            {
                // 获取真实的key
                NSString *key = mapDict[name];
                value = dict[key];
            }
        }
        [obj setValue:value forKey:name];

//        NSLog(@"%@",name);
    }
    return obj;
}
```

- 使用方法，要初入一个冲突的字典

```objc
SLQHtmlItem *item  = [SLQHtmlItem objectWithDict:dict mapDict:@{@"ID":@"id"}];
```

## 17、控制器的父子关系

- 1、控制器父子关系的建立原则
    - 如果2个控制器的view是父子关系(不管是直接还是间接的父子关系)，那么这2个控制器也应该为父子关系
    `[self.view addSubview:view];`
    `[self addChildViewController:viewController];`

- 2、获得所有的子控制器
`@property(nonatomic,readonly) NSArray *childViewControllers;`
- 3、添加一个子控制器

```objc
// ViewController成为了self的子控制器
// self成为了ViewController的父控制器
// 通过addChildViewController添加的控制器都会存在于childViewControllers数组中
[selfaddChildViewController:[[ViewControlleralloc] init]];
```

- 4、获得父控制器
`@property(nonatomic,readonly) UIViewController *parentViewController;`
- 5、将一个控制器从它的父控制器中移除
`// 控制器a从它的父控制器中移除`
`[selfremoveFromParentViewController];`
- 6、添加控件到主控件后调用

```objc
- (void)didMoveToParentViewController:(UIViewController *)parent
{
}
```

## 18、UIScrollView使用注意

- UIScrollView内部子控件添加约束的注意点
- 1、子控件的尺寸不能通过UIScrollView来计算，可以考虑通过以下方式计算
    - 可以设置固定值（width==100，height==300）
    - 可以相对于UIScrollView以外的其他控件来计算尺寸
- 2、UIScrollView的frame应该通过`子控件以外的其他控件来计算
- 3、UIScrollView的contentSize通过子控件来计算
    - 根据子控件的尺寸以及子控件与UIScrollView之间的间距

## 19、窗口悬停

![](../images/窗口悬停.gif)

### 不使用自动布局,使用frame
- 添加几个子控件（UIScrollView ->UIImageView，UIView）,直接设置frame确定尺寸

- 1、分析
    - 这个UIView在达到窗口顶部时一直显示在窗口顶部，但是UIScrollView可以继续向上滚动
    - 监视UIScrollView的偏移量，当view达到顶部时，将view从UIScrollView中移除，添加到self.view,向下滑动时，到这个原来view的位置时再把view添加到UIScrollView，再向下拖动就方法图片
    - 实现时要用一个属性来记录view 在UIScrollView中得位置
    - 注意，这里通过storyboard添加的子控件，没有使用自动布局，所以要把自动布局完毕关闭，不然会出现莫名其妙的问题
- 2、实现
    - 主要是在滚动scrollView时进行判断

```objc
- (void)viewDidLoad {
    [superviewDidLoad];
   // 设置scrollView的尺寸
    _blueScrollView.contentSize = CGSizeMake(0, 1000);
    // 记录redView旧值
    _oldRect = self.redView.frame;
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    // 滚动过程中判断红色view 的位置，如果位于最顶部，就从UIScrollView脱离，添加到self.view，否则就添加到UIScrollView
    // 当前view距离顶部的距离
    CGFloat height = scrollView.contentOffset.y - self.imageView.bounds.size.height;

    if (height >= 0) {

        CGRect temp = _oldRect;
        temp.origin.y = 0; // 设置新的位置
        self.redView.frame = temp;
        [self.view addSubview:self.redView];
    }
    else
    {
        self.redView.frame = _oldRect; // 还原
        [self.blueScrollViewaddSubview:self.redView];
    }
    // 缩放图片
    CGFloat scale = (1 - scrollView.contentOffset.y/70);
    NSLog(@"%f",scale);
    scale = scale >= 1 ? scale : 1;
    self.imageView.transform = CGAffineTransformMakeScale(scale, scale);
}
```
### 自动布局实现

- 自动布局实现貌似又复杂了，使用自动布局的话，不能直接修改frame，否则会出问题,所以这里新建一个大小一致的view，显示到self.view，
— 首先要重新生成一个UIView，最好就是克隆一个和redView一样的view，然后设置在原始位置，滚动到顶部时显示，向下滚动时隐藏。
- 代码如下：

```objc
- (void)viewDidLoad {
    [superviewDidLoad];
    // 新建一个view，大小和redView一样，显示在顶部
    _stopView = [[UIView alloc ]init];
    _stopView.frame = self.redView.bounds;
    _stopView.backgroundColor = self.redView.backgroundColor;
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    // 滚动过程中判断红色view 的位置，如果位于最顶部，就从UIScrollView脱离，添加到self.view，否则就添加到UIScrollView
    // 使用自动布局的话，不能直接修改frame，否则会出问题,所以这里新建一个大小一致的view，显示到self.view，

    // 当前view距离顶部的距离
    CGFloat height = self.imageView.bounds.size.height - scrollView.contentOffset.y;

    if (height <= 0) {
        // 显示
        _stopView.hidden = NO;
        [self.view addSubview:_stopView];
    }else
    {
        // 隐藏
        _stopView.hidden = YES;
        [self.blueScrollViewaddSubview:self.redView];
    }

    // 缩放图片
    CGFloat scale = (1 - scrollView.contentOffset.y/70);
    scale = scale >= 1 ? scale : 1;
    self.imageView.transform = CGAffineTransformMakeScale(scale, scale);
}
```

## 20、const和指针以及数组

- 指针与const

```objc
void pointAndConst()
{
    int num  = 10;
    int bad = 20;
    const int *p1 ;
    int const *p2 ;
    int *const p3 ;

    // 常量指针 --- 是一个指针，由一个不可变指针指向一个变量
    constint *p = &num; // 修饰的是*p,所以直接通过*p修改数据不允许
    // int const *p = &num; // 修饰的也是*p,所以直接通过*p修改数据不允许
    // 指针常量 --- 是一个常量，由一个可变指针指向一个常量
    //int * const p = &num; // 修饰的是p，所以直接通过p修改数据不允许
    // 总结：const右边修饰的是神马，那这个就不可变
    NSLog(@"%d--%p",num,&num);
    //        *p += 10;
    //        num += 10;
    NSLog(@"%d--%p",*p,p);

    p = &bad; //
    NSLog(@"%d--%p",*p,p);
}
```
- 总结：const右边修饰的是神马，那这个神马就不可变（p，*p）

- 指针和数组

- 一维数组分析

```objc
/*
 * 一维数组
 */
int numbers[4] = {11,21,31,41};
// numbers 是数组的地址，&number等价与一个指向数组的指针，如果进行+1的话，指针跳转的时整个数组的长度（16个字节）
// 0x7fff5fbff780--0x7fff5fbff790
NSLog(@"%p--%p",&numbers,&numbers + 1);
// numbers 是数组的地址，等价于首元素的地址number[0],首元素是一个int类型的指针，故+1，跳转4个字节
 NSLog(@"%p--%p",numbers,numbers + 1);
// numbers[0] 是数组的首元素，&numbers[0]等价一个int类型的数据，+1的话跳转一个int类型大小（4个字节）
// 0x7fff5fbff780--0x7fff5fbff784
NSLog(@"%p--%p--%d",&numbers[0],&numbers[0] + 3,*(&numbers[0] + 3));
三维数组分析

/*
* 三维数组
*/
int numbers[3][3] = {
    {11,12,13}, // numbers[0]
    {33,34,35}, // numbers[1]
    {55,56,57}  // numbers[2]
};
// numbers 是数组的地址，&number等价与一个指向数组的指针，如果进行+1的话，指针跳转的时整个数组的长度（36个字节）
// 0x7fff5fbff770--0x7fff5fbff794
NSLog(@"%p--%p--%d",&numbers,&numbers + 1,***(&numbers + 1)); // 数组越界

// numbers 是数组的地址，等价与number[0],而numbers[0]是一个指向{11，22，33}一维数组的指针, +1指针跳转数组大小字节（12字节））
NSLog(@"%p--%p--%d",numbers,numbers + 1,**(numbers + 1));

// numbers[0] 是{11，22，33}数组的地址，&numbers[0]等价与一个指向{11，22，33}数组的指针，+1的话，指针跳转数组大小字节（12字节）
// 0x7fff5fbff770--0x7fff5fbff77c
// numbers[0] 是数组{11,22,33}的地址，等价于numbers[0][0],+1的话跳转一个int类型大小（4个字节）
NSLog(@"%p--%p--%d",numbers[0],numbers[0] + 1,*(numbers[0] + 1));
NSLog(@"---------------");

// &numbers[0] 等价与指向一维数组{11,22,33}的指针,+1的话跳转数组大小（12个字节）
NSLog(@"%p--%p--%d",&numbers[0],&numbers[0] + 1,**(&numbers[0] + 1));

// &numbers[0][0] 等价与一个指向int类型的指针，+1跳转4个字节
 NSLog(@"%p--%p--%d",&numbers[0][0],&numbers[0][0] + 3,*(&numbers[0][0] + 3));

// &numbers[1] 等价与指向一维数组{33,34,35}的指针,+1的话跳转数组大小（12个字节）
NSLog(@"%p--%p--%d",&numbers[1],&numbers[1] + 1,**(&numbers[1]+ 2));
```
- 总结

- 1、指针p的加减法运算
    - 指针p + N
        - p里面存储的地址值 + N * 指针所指向类型的占用字节数
    - 指针p - N
        - p里面存储的地址值 - N * 指针所指向类型的占用字节数
- 2、数组名
    - 存储的是数组首元素的地址
    - 等价于:一个指向数组首元素的指针
    - 数组名 + 1 的跨度：数组首元素的占用字节数
- 3、其他结论
    - &num + 1的跨度：num的占用字节数



http://www.cnblogs.com/songliquan/
