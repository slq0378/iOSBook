# UITableView总结1


## UITableView

- tableView如何显示数据
    - 设置dataSource数据源
    - 数据源要遵守UITableViewDataSource协议
    - 数据源要实现协议中的某些方法

```objc
/**
 *  告诉tableView一共有多少组数据
 */
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView

/**
 *  告诉tableView第section组有多少行
 */
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section

/**
 *  告诉tableView第indexPath行显示怎样的cell
 */
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath

/**
 *  告诉tableView第section组的头部标题
 */
- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section

/**
 *  告诉tableView第section组的尾部标题
 */
- (NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section
```


## tableView性能优化 - cell的循环利用方式1

```objc
/**
 *  什么时候调用：每当有一个cell进入视野范围内就会调用
 */
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 0.重用标识
    // 被static修饰的局部变量：只会初始化一次，在整个程序运行过程中，只有一份内存
    static NSString *ID = @"cell";

    // 1.先根据cell的标识去缓存池中查找可循环利用的cell
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    // 2.如果cell为nil（缓存池找不到对应的cell）
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:ID];
    }

    // 3.覆盖数据
    cell.textLabel.text = [NSString stringWithFormat:@"testdata - %zd", indexPath.row];

    return cell;
}
```
## tableView性能优化 - cell的循环利用方式2
- 定义一个全局变量

```objc
// 定义重用标识
NSString *ID = @"cell";
```

- 注册某个标识对应的cell类型

```objc
// 在这个方法中注册cell
- (void)viewDidLoad {
    [super viewDidLoad];

    // 注册某个标识对应的cell类型
    [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:ID];
}
```

- 在数据源方法中返回cell

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 1.去缓存池中查找cell
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    // 2.覆盖数据
    cell.textLabel.text = [NSString stringWithFormat:@"testdata - %zd", indexPath.row];

    return cell;
}
```

## tableView性能优化 - cell的循环利用方式3

- 在storyboard中设置UITableView的Dynamic Prototypes Cell
![](../images/Snip20150602_152.png)

- 设置cell的重用标识
![](../images/Snip20150602_153.png)

- 在代码中利用重用标识获取cell

```objc
// 0.重用标识
// 被static修饰的局部变量：只会初始化一次，在整个程序运行过程中，只有一份内存
static NSString *ID = @"cell";

// 1.先根据cell的标识去缓存池中查找可循环利用的cell
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

// 2.覆盖数据
cell.textLabel.text = [NSString stringWithFormat:@"cell - %zd", indexPath.row];

return cell;
```
## 错误将UIViewController当做UITableViewController来用

![](../images/Snip20150602_110.png)

## UITableView的常见设置

```objc
// 分割线颜色
self.tableView.separatorColor = [UIColor redColor];

// 隐藏分割线
self.tableView.separatorStyle = UITableViewCellSeparatorStyleNone;
```

## UITableViewCell的常见设置

```objc
// 取消选中的样式
cell.selectionStyle = UITableViewCellSelectionStyleNone;
// 设置选中的背景色
UIView *selectedBackgroundView = [[UIView alloc] init];
selectedBackgroundView.backgroundColor = [UIColor redColor];
cell.selectedBackgroundView = selectedBackgroundView;

// 设置默认的背景色
cell.backgroundColor = [UIColor blueColor];

// 设置默认的背景色
UIView *backgroundView = [[UIView alloc] init];
backgroundView.backgroundColor = [UIColor greenColor];
cell.backgroundView = backgroundView;

// backgroundView的优先级 > backgroundColor
// 设置指示器
//    cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
cell.accessoryView = [[UISwitch alloc] init];
```

## 自定义cell

- `等高的cell`
    - `storyboard自定义cell`
        - 1.创建一个继承自UITableViewCell的子类，比如XMGDealCell<br>
        ![](../images/Snip20150602_305.png)
        - 2.在storyboard中
            - 往cell里面增加需要用到的子控件<br>
            ![](../images/Snip20150602_302.png)
            - 设置cell的重用标识<br>
            ![](../images/Snip20150602_303.png)
            - 设置cell的class为XMGDealCell<br>
            ![](../images/Snip20150602_304.png)
        - 3.在控制器中
            - 利用重用标识找到cell
            - 给cell传递模型数据<br>
            ![](../images/Snip20150602_301.png)
        - 4.在XMGDealCell中
            - 将storyboard中的子控件连线到类扩展中<br>
            ![](../images/Snip20150602_299.png)
            - 需要提供一个模型属性，重写模型的set方法，在这个方法中设置模型数据到子控件上<br>
            ![](../images/Snip20150602_298.png)
            ![](../images/Snip20150602_300.png)
    - `xib自定义cell`
        - 1.创建一个继承自UITableViewCell的子类，比如XMGDealCell<br>
        - 2.创建一个xib文件（文件名建议跟cell的类名一样），比如XMGDealCell.xib
            - 拖拽一个UITableViewCell出来
            - 修改cell的class为XMGDealCell
            - 设置cell的重用标识
            - 往cell中添加需要用到的子控件
        - 3.在控制器中
            - 利用registerNib...方法注册xib文件
            - 利用重用标识找到cell（如果没有注册xib文件，就需要手动去加载xib文件）
            - 给cell传递模型数据<br>
        - 4.在XMGDealCell中
            - 将xib中的子控件连线到类扩展中
            - 需要提供一个模型属性，重写模型的set方法，在这个方法中设置模型数据到子控件上
            - 也可以将创建获得cell的代码封装起来（比如cellWithTableView:方法)
    - `代码自定义cell(使用frame)`
        - 1.创建一个继承自UITableViewCell的子类，比如XMGDealCell
            - 在initWithStyle:reuseIdentifier:方法中
                - 添加子控件
                - 设置子控件的初始化属性（比如文字颜色、字体）
            - 在layoutSubviews方法中设置子控件的frame
            - 需要提供一个模型属性，重写模型的set方法，在这个方法中设置模型数据到子控件
        - 2.在控制器中
            - 利用registerClass...方法注册XMGDealCell类
            - 利用重用标识找到cell（如果没有注册类，就需要手动创建cell）
            - 给cell传递模型数据
            - 也可以将创建获得cell的代码封装起来（比如cellWithTableView:方法）
    - `代码自定义cell(使用autolayout)`
        - 1.创建一个继承自UITableViewCell的子类，比如XMGDealCell
            - 在initWithStyle:reuseIdentifier:方法中
                - 添加子控件
                - 添加子控件的约束（建议使用`Masonry`）
                - 设置子控件的初始化属性（比如文字颜色、字体）
            - 需要提供一个模型属性，重写模型的set方法，在这个方法中设置模型数据到子控件
        - 2.在控制器中
            - 利用registerClass...方法注册XMGDealCell类
            - 利用重用标识找到cell（如果没有注册类，就需要手动创建cell）
            - 给cell传递模型数据
            - 也可以将创建获得cell的代码封装起来（比如cellWithTableView:方法）
- `非等高的cell`
    - xib自定义cell
    - storyboard自定义cell
    - 代码自定义cell（frame）
    - 代码自定义cell（Autolayout）

- 在SLQFengCell中

## 代理模式

* 代理设计模式的作用:
    * 1.A对象监听B对象的一些行为，A成为B的代理
    * 2.B对象想告诉A对象一些事情，A成为B的代理

* 代理设计模式的总结：
    * 如果你想监听别人的一些行为，那么你就要成为别人的代理
    * 如果你想告诉别人一些事情，那么就让别人成为你的代理

* 代理设计模式的开发步骤
    * 1.拟一份协议（协议名字的格式：控件名 + Delegate），在协议里面声明一些代理方法（一般代理方法都是@optional）
    * 2.声明一个代理属性：@property (nonatomic, weak) id<代理协议> delegate;
    * 3.在内部发生某些行为时，调用代理对应的代理方法，通知代理内部发生什么事
    * 4.设置代理：xxx.delegate = yyy;
    * 5.yyy对象遵守协议，实现代理方法

## 代理和通知的区别
- 代理：1个对象只能告诉另1个对象发生了什么事
- 通知：1个对象可以告诉N个对象发生了什么事

## KVC\KVO
- KVC(Key Value Coding)常见作用：给模型属性赋值
- KVO(Key Value Observing)常用作用：监听模型属性值的改变
- KVO的使用步骤<br>

```objc
// cc监听了aa的name属性的改变
[aa addObserver:cc forKeyPath:@"name" options: NSKeyValueObservingOptionOld context:nil];

// cc得实现监听方法
/**
 * 当监听到object的keyPath属性发生了改变
 */
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    NSLog(@"监听到%@对象的%@属性发生了改变， %@", object, keyPath, change);
}
```


## 一、UITableView的常用属性

- 1、分割线

`// 分割线`
`self.tableView.separatorColor = [UIColorredColor];`
![](../images/屏幕快照 2015-06-04 10.57.43.png)

`// 隐藏分割线`
`self.tableView.separatorStyle = UITableViewCellSeparatorStyleNone;`
![](../images/屏幕快照 2015-06-04 10.58.05.png)

- 2、选中样式

`// cell选中样式，貌似几个枚举效果都一样`
`cell.selectionStyle = UITableViewCellSelectionStyleBlue;`
![](../images/屏幕快照 2015-06-04 11.01.58.png)

`// cell选中背景色`
`UIView *selectedBackground = [[UIView alloc] init];`
`selectedBackground.backgroundColor = [UIColor greenColor];`
`cell.selectedBackgroundView = selectedBackground;`
![](../images/屏幕快照 2015-06-04 11.04.50.png)

- 3、背景色

`// cell默认背景色`
`cell.backgroundColor = [UIColor redColor];`
![](../images/屏幕快照 2015-06-04 11.10.38.png)

```objc
// backgroundView 的优先级大于backgroundColor
UIView *background = [[UIView alloc] init];
background.backgroundColor = [UIColor orangeColor];
cell.backgroundView = background;
```
![](../images/屏幕快照 2015-06-04 11.10.58.png)

- 设置背景色有两种方式，注意一点就是 :
    - backgroundView 的优先级大于backgroundColor

- 4、指示器
    - 指示器默认显示的是第一个 UITableViewCellAccessoryNone

```objc
    typedef NS_ENUM(NSInteger, UITableViewCellAccessoryType) {
    UITableViewCellAccessoryNone,                   // don't show any accessory view
    UITableViewCellAccessoryDisclosureIndicator,    // regular chevron. doesn't track
    UITableViewCellAccessoryDetailDisclosureButton, // info button w/ chevron. tracks
    UITableViewCellAccessoryCheckmark,              // checkmark. doesn't track
    UITableViewCellAccessoryDetailButton NS_ENUM_AVAILABLE_IOS(7_0) // info button. tracks
};
```
样式依次为：
![](../images/屏幕快照 2015-06-04 11.18.42.png)
![](../images/屏幕快照 2015-06-04 11.15.17.png)
![](../images/屏幕快照 2015-06-04 11.17.19.png)
![](../images/屏幕快照 2015-06-04 11.14.33.png)
![](../images/屏幕快照 2015-06-04 11.14.56.png)

`// cell指示器`
`cell.accessoryType = UITableViewCellAccessoryCheckmark;`

- 也可以自定义View显示。

`// 显示预定义按钮`
`cell.accessoryView = [UIButtonbuttonWithType:UIButtonTypeContactAdd];`
![](../images/屏幕快照 2015-06-04 11.41.00.png)

```objc
    // 自定义view
    UILabel *name  = [[UILabel alloc] init];
    name.frame = CGRectMake(200, 0, 80, 40);
    name.text = [NSString stringWithFormat: @"英雄%zd",indexPath.row + 1];
    name.textAlignment = NSTextAlignmentRight;
    name.textColor = [UIColor lightGrayColor];
    cell.accessoryView = name;
```
![](../images/屏幕快照 2015-06-04 11.43.17.png)

## 二、cell的循环利用方式

- 1、方式1

```objc
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
    {
    /**
     *  什么时候调用：每当有一个cell进入视野范围内就会调用
     */
    // 0.重用标识
    // 被static修饰的局部变量：只会初始化一次，在整个程序运行过程中，只有一份内存
    static NSString *ID = @"cell";

    // 1.先根据cell的标识去缓存池中查找可循环利用的cell
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    // 2.如果cell为nil（缓存池找不到对应的cell）
    if (cell == nil) {
        cell = [[UITableViewCellalloc] initWithStyle:UITableViewCellStyleDefaultreuseIdentifier:ID];
    }

    // 3.覆盖数据
    cell.textLabel.text = [NSString stringWithFormat:@"testdata - %zd", indexPath.row];


    return  cell;
    }
```

- 2、方式2

- 定义一个全局变量

```objc
   // 定义重用标识
     NSString *ID = @"cell";
```

- 注册某个标识对应的cell类型

```objc
  - (void)viewDidLoad {
        [super viewDidLoad];

        // 注册某个标识对应的cell类型
        [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:ID];
    }
```

- 在数据源方法中返回cell,
dequeueReusableCellWithIdentifier:方法会自动查找cell，如果没有找到，就按照注册类型的cell，重新创建一个并返回。

```objc
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
    {
        // 1.去缓存池中查找cell
        UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

        // 2.覆盖数据
        cell.textLabel.text = [NSString stringWithFormat:@"testdata - %zd", indexPath.row];

        return cell;
    }
```

- 3、方式3

- 在storyboard中设置UITableView的Dynamic Prototypes Cell

![](../images/Snip20150602_152.png)

- 设置cell的重用标识

![](../images/Snip20150602_153.png)

- 在代码中利用重用标识获取cell

```objc
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
    {

    // 0.重用标识
    // 被static修饰的局部变量：只会初始化一次，在整个程序运行过程中，只有一份内存
    static NSString *ID = @"cell";

    // 1.先根据cell的标识去缓存池中查找可循环利用的cell
     UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    // 2.覆盖数据
    cell.textLabel.text = [NSString stringWithFormat:@"cell - %zd", indexPath.row];

    return  cell;
    }
```

## 三、自定义cell

- 使用系统自带的cell有时不能满足需求，可以自定义cell，实现自己想要的界面。

- cell的自定义分两种，等高的和不等高的，等高的比较容易实现。

- 实现界面如下：

![](../images/屏幕快照 2015-06-04 15.07.43.png)

## 1、等高cell

- storyboard实现自定义cell

- 创建一个继承自UITableViewCell的子类，比如SLQFengCell

```objc
    #import <UIKit/UIKit.h>

    @interface SLQFengCell : UITableViewCell

    @end
```

- 在storyboard中
    - 往cell里面增加需要用到的子控件
    - 设置cell的重用标识
    - 设置cell的class为SLQFengCell
- 在控制器中
    - 利用重用标识找到cell
     -给cell传递模型数据

```objc
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
    {
    // 从缓存池中获取数据，如果没有就根据ID创建一个
    static NSString *ID =@"feng";
    SLQFengCell *cell = [tableView     dequeueReusableCellWithIdentifier:ID];
    // 传递模型数据
    cell.fengNews = self.news[indexPath.row];
    return cell;
    }
```

- 在SLQFengCell中
    - 将storyboard中的子控件连线到类扩展中

```objc
    @interfaceSLQFengCell ()
    @property (weak, nonatomic) IBOutletUIImageView *image;
    @property (weak, nonatomic) IBOutletUILabel *titleLable;
    @property (weak, nonatomic) IBOutletUILabel *topicLable;
    @property (weak, nonatomic) IBOutletUILabel *commentLable;
    @end
```

- 需要提供一个模型属性，重写模型的set方法，在这个方法中设置模型数据到子控件上
```objc
    #import <UIKit/UIKit.h>

    @classSLQFengNews;

    @interface SLQFengCell : UITableViewCell
    @property (strong, nonatomic) SLQFengNews *fengNews; // 模型属性
    @end
```

- 重写set方法，对模型对象就行赋值。

```objc
    // 重写set方法
    - (void)setFengNews:(SLQFengNews *)fengNews
    {
    _fengNews = fengNews;
    // 设置图片
    self.image.image = [UIImage imageNamed:fengNews.image];
    // 设置标题
    self.titleLable.text = fengNews.title;
    self.titleLable.font = [UIFontsystemFontOfSize:15];
    // 判断是否是专题
    if (fengNews.isTopic)
    {
        self.topicLable.hidden = NO;
        self.topicLable.text = @"专题";
        self.topicLable.font = [UIFontboldSystemFontOfSize:12];
    }
    else
    {
        self.topicLable.hidden = YES;
    }
    // 设置评论数
    self.commentLable.text = fengNews.comment;
    self.commentLable.font = [UIFontsystemFontOfSize:12];
    }
```

- 在模型类中添加对应数量的属性，并写一个对象方法返回模型初始化后的对象

```objc
    #import <Foundation/Foundation.h>
    @interface SLQFengNews : NSObject
    /*图片*/
    @property (strong, nonatomic) NSString *image;
    /*标题*/
    @property (strong, nonatomic) NSString *title;
    /*专题*/
    @property (assign, nonatomic,getter = isTopic) BOOL topic;
    /*评论数*/
    @property (strong, nonatomic) NSString *comment;
    + (instancetype)fengNewsWithDict:(NSDictionary *)dict;
    @end


    //实现文件
    #import "SLQFengNews.h"
    @implementation SLQFengNews
    // 根据字典返回对象
    + (instancetype)fengNewsWithDict:(NSDictionary *)dict
    {
        SLQFengNews *feng = [[SLQFengNewsalloc] init];
        [feng setValuesForKeysWithDictionary:dict];
        return  feng;
    }
    @end
```

- 数据使用plist文件，保存各种信息


## xib自定义cell

- 很多步骤和在storyboard里差不多
    - 1.创建一个继承自UITableViewCell的子类，比如 SLQFengCell （同上）
    - 2.创建一个xib文件（文件名建议跟cell的类名一样），比如SLQFengCell.xib

- 拖拽一个UITableViewCell出来
    - 修改cell的class为 SLQFengCell （同上）
    - 设置cell的重用标识  （同上）
    - 往cell中添加需要用到的子控件，实现如下界面

![](../images/屏幕快照 2015-06-04 16.30.12.png)

- 3.在控制器中
    - 利用registerNib...方法注册xib文件
    - 利用重用标识找到cell（如果没有注册xib文件，就需要手动去加载xib文件）
    - 给cell传递模型数据，代码如下，这里对cell的创建方法进行封装

```objc
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
    {
        SLQFengCell *cell = [SLQFengCell cellWithTableView:tableView]; // 这里对cell的创建进行封装
        // 传递模型数据
        cell.fengNews = self.news[indexPath.row];
         return cell;
    }
```

- 4.在SLQFengCell中
    -将xib中的子控件连线到类扩展中（同上）
    -需要提供一个模型属性，重写模型的set方法，在这个方法中设置模型数据到子控件上（同上）
    -也可以将创建获得cell的代码封装起来（比如cellWithTableView:方法）

- 在 SLQFengCell类中定义方法cellWithTableView

```objc
    // 返回cell对象，这里做加载xib的操作
    + (instancetype)cellWithTableView:(UITableView *)tableView
    {
    static NSString *ID = @"feng";
        SLQFengCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];
    if(cell == nil)
        {
            cell = [[[NSBundlemainBundle] loadNibNamed:NSStringFromClass([SLQFengCellclass]) owner:niloptions:nil] lastObject];
        }
    return cell;
    }
```

## 代码自定义cell(使用frame)

- 1.创建一个继承自UITableViewCell的子类，比如XMGDealCell
    -在initWithStyle:reuseIdentifier:方法中
    -添加子控件
    - 设置子控件的初始化属性（比如文字颜色、字体）

```objc
    // 初始化控件，添加控件到contentView，并设置相关的属性
    - (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier
    {
        if (self = [super initWithStyle:style reuseIdentifier:reuseIdentifier])
        {
            UIImageView *image = [[UIImageView alloc] init];
            [self.contentView addSubview:image];
            self.image = image;

            UILabel *title  = [[UILabel alloc] init];
            title.numberOfLines = 0;
            [self.contentView addSubview:title];
            self.titleLable = title;

            UILabel *topic  = [[UILabel alloc] init];
            topic.textColor = [UIColor redColor];
            topic.font = [UIFont systemFontOfSize:12];
            [self.contentView addSubview:topic];
            self.topicLable = topic;

            UILabel *comment  = [[UILabel alloc] init];
            comment.font = [UIFont systemFontOfSize:12];
            comment.textColor = [UIColor blueColor];
            comment.textAlignment = NSTextAlignmentRight;
            [self.contentView addSubview:comment];
            self.commentLable = comment;
        }
        return  self;
    }
```

- 在layoutSubviews方法中设置子控件的frame

```objc
    // 设置控件的frame
    - (void)layoutSubviews
    {
        [superlayoutSubviews];

        CGFloat contentH = self.contentView.frame.size.height;
        CGFloat contentW = self.contentView.frame.size.width;
        CGFloat margin = 10;

        CGFloat imageX = margin;
        CGFloat imageY = margin;
        CGFloat imageW = 50;
        CGFloat imageH = contentH - 2 * imageY;
        self.image.frame = CGRectMake(imageX, imageY, imageW, imageH);

        // titleLabel
        CGFloat titleX = CGRectGetMaxX(self.image.frame) + margin;
        CGFloat titleY = imageY;
        CGFloat titleW = contentW - titleX - margin;
        CGFloat titleH = 20;
        self.titleLable.frame = CGRectMake(titleX, titleY, titleW, titleH);

        // topicLabel
        CGFloat topicX = titleX;
        CGFloat topicH = 20;
        CGFloat topicY = contentH - margin - topicH;
        CGFloat topicW = 70;
        self.topicLable.frame = CGRectMake(topicX, topicY, topicW, topicH);

        // commentLable
        CGFloat commentH = topicH;
        CGFloat commentY = topicY;
        CGFloat commentX = CGRectGetMaxX(self.topicLable.frame) + margin;
        CGFloat commentW = contentW - commentX - margin;
        self.commentLable.frame = CGRectMake(commentX, commentY, commentW, commentH);
    }
```
- 需要提供一个模型属性，重写模型的set方法，在这个方法中设置模型数据到子控件


- 2.在控制器中

- 利用registerClass...方法注册SLQFengCell类

```objc
    - (void)viewDidLoad {
        [superviewDidLoad];
        [self.tableViewregisterClass:[SLQFengCellclass] forCellReuseIdentifier:@"feng"];
    }
```
- 利用重用标识找到cell（如果没有注册类，就需要手动创建cell）

- 给cell传递模型数据 (同上)

- 也可以将创建获得cell的代码封装起来（比如cellWithTableView:方法） (同上)

- 代码参考

 [](http://pan.baidu.com/s/1pJ7Oayn)

## 代码自定义cell(使用autolayout)

- 1.创建一个继承自UITableViewCell的子类，比如XMGDealCell
    - 在initWithStyle:reuseIdentifier:方法中
    - 添加子控件
    - 添加子控件的约束（masonry）（这里添加约束，layoutSubviews不用重写了）

```objc
    // 初始化控件，添加控件到contentView，并设置相关的属性
    - (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier
    {
        if (self = [super initWithStyle:style reuseIdentifier:reuseIdentifier])
        {
            UIImageView *image = [[UIImageView alloc] init];
            [self.contentView addSubview:image];
            self.image = image;

            UILabel *title  = [[UILabel alloc] init];
            title.numberOfLines = 0;
            [self.contentView addSubview:title];
            self.titleLable = title;

            UILabel *topic  = [[UILabel alloc] init];
            topic.textColor = [UIColor redColor];
            topic.font = [UIFont systemFontOfSize:12];
            [self.contentView addSubview:topic];
            self.topicLable = topic;

            UILabel *comment  = [[UILabel alloc] init];
            comment.font = [UIFont systemFontOfSize:12];
            comment.textColor = [UIColor blueColor];
            comment.textAlignment = NSTextAlignmentRight;
            [self.contentView addSubview:comment];
            self.commentLable = comment;


            // 添加约束
            CGFloat margin = 10;
            [self.image makeConstraints:^(MASConstraintMaker *make) {
                make.size.equalTo(50);
                make.top.left.equalTo(self.contentView.top).offset(margin);
            }];

            [self.titleLable makeConstraints:^(MASConstraintMaker *make) {
                make.left.equalTo(self.image.right).offset(margin);
                make.top.equalTo(self.image.top);
                make.right.equalTo(self.contentView.right).offset(-margin);
            }];

            [self.topicLable makeConstraints:^(MASConstraintMaker *make) {
                make.left.equalTo(self.titleLable.left);
                make.bottom.equalTo(self.image.bottom);
                make.width.equalTo(100);
            }];

            [self.commentLable makeConstraints:^(MASConstraintMaker *make) {
                make.left.equalTo(self.topicLable.right).offset(margin);
                make.right.equalTo(self.titleLable.right);
                make.bottom.equalTo(self.topicLable.bottom);
            }];

        }
        return  self;
    }
```

- 设置子控件的初始化属性（比如文字颜色、字体）
- 需要提供一个模型属性，重写模型的set方法，在这个方法中设置模型数据到子控件

- 2.在控制器中 (同上)
-利用registerClass...方法注册SLQFengCell类

```objc
    - (void)viewDidLoad {
        [superviewDidLoad];
        [self.tableViewregisterClass:[SLQFengCellclass] forCellReuseIdentifier:@"feng"];
    }
```

- 利用重用标识找到cell（如果没有注册类，就需要手动创建cell）
- 给cell传递模型数据 (同上)
- 也可以将创建获得cell的代码封装起来（比如cellWithTableView:方法） (同上)

## 自定义非等高的cell

- 如常见的微博界面，有的微博只有文字，有的有文字和图片。这些微博的高度不固定需要重新计算。

- 这里简单说一下几种方法。前面的步骤和设置等高的cell一样。现在来说说不一样的地方。

- 效果如下：

![](../images/屏幕快照 2015-06-05 22.33.08.png)

- 1、在storyboard\xib里实现如下界面
    - 使用自动布局添加约束,在xib里创建和在storyboard非常类似。

![](../images/屏幕快照 2015-06-05 22.28.34.png)

- 2、计算cell高度
- 2.1、在模型类中给每个cell添加一个高度属性

```objc
    // cell高度
    @property (assign, nonatomic) CGFloat cellHeight;
```
- 2.2、设置UILable的宽度

```objc
    //UILable的宽度，必须进行设置，不然计算时就出问题

    - (void)awakeFromNib

    {

        // 设置label每一行文字的最大宽度

        // 为了保证计算出来的数值跟真正显示出来的效果一致

        self.contentLable.preferredMaxLayoutWidth = [[UIScreenmainScreen] bounds].size.width -20;

     }
```

- 2.3、在cell的封装类计算cell高度cellHeight
    - 这里需要注意的地方就是强制布局，因为在设置过数据后，每一个控件虽然都有数据了，但是数据并没有立即显示到tableView中，所以高度值是不准确的，主要是因为UILable的高度没计算出来。这里调用方法layoutIfNeeded                强制进行布局，也就是强制tableView进行更新数据，然后里面的UILable的高度就会有系统计算出来，高度值就是正确的高度值了。

```objc
    // 强制布局
    [self layoutIfNeeded];

    // 计算高度
    if (self.pictureImage.hidden)
    {
         weibo.cellHeight = CGRectGetMaxY(self.contentLable.frame) + 10;
    }
    else
    {
         weibo.cellHeight = CGRectGetMaxY(self.pictureImage.frame) + 10;
    }
```
- 2.4、实现两个代理方法
    - 关键是 estimatedHeightForRowAtIndexPath 的实现，实现了这个，上边的强制布局才会有效。

```objc
    /**
     *  // 设置cell高度
     */
    - (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
    {
        SLQWeibo *weibo = self.weibos[indexPath.row];
        return  weibo.cellHeight;
    }
    /**
     * 返回每一行的估计高度
     * 只要返回了估计高度，那么就会先调用tableView:cellForRowAtIndexPath:方法创建cell，再调   用tableView:heightForRowAtIndexPath:方法获取cell的真实高度
     */
    - (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath
    {
        return200; // 不要设置的太小
    }
```

- 3、UITableView的添加、删除和更新
    -  UITableView的数据进行修改时要手动对UITableView进行刷新。

- 3.1、添加一个工具栏以及三个按钮，并拖线实现单击事件

```objc
    - (IBAction)addBtn:(id)sender;
    - (IBAction)updateBtn:(id)sender;
    - (IBAction)deleteBtn:(id)sender;
```

- 3.2、添加一行或者多行
    - 对cell的修改，直接修改模型数据就行，UITableView的数据源会自动获取数据并显示到cell中。

```objc
// 添加按钮
- (IBAction)addBtn:(id)sender
{
    // 添加到模型
    SLQWeibo *weibo = [[SLQWeibo alloc] init];
    weibo.name = @"天高皇帝远";
    weibo.vip = NO;
    weibo.text = @"上联：做 I T 风风雨雨又一年;\n下联：卖电脑辛辛苦苦每一天 ;\n横批：从小不学好，长大卖电脑！";
    weibo.icon = [NSStringstringWithFormat:@"01%d.png",arc4random_uniform(9)];
    // 添加到数组
    [self.weibosinsertObject:weibo atIndex:0]; // 添加到数组

    // 刷新表格
    //[self.tableView reloadData];
    // 带有的的动画的删除方式，不过需要指定参数
    [self.tableViewinsertRowsAtIndexPaths:@[[NSIndexPathindexPathForItem:0inSection:0]]withRowAnimation:UITableViewRowAnimationRight];

}
```

- 3.3、更新某一行cell的数据

```objc
// 更新按钮
- (IBAction)updateBtn:(id)sender
{
    SLQWeibo *cell = self.weibos[2];
    cell.text = @"一天和同学出去吃饭，买单的时候想跟服务员开下玩笑。\n“哎呀，今天没带钱出来啊。”\n“你可以刷卡。”\n“可是我也没带卡出来的啊。”\n“那你可以刷碗！”";

    // 刷新表格
    [self.tableViewreloadData];
    // 带有的的动画的删除方式，需要指定参数
    //[self.tableView reloadRowsAtIndexPaths:@[[NSIndexPath indexPathForItem:2 inSection:0]] withRowAnimation:UITableViewRowAnimationFade];

}
```

- 3.4、删除一行或者多行

```objc
// 删除按钮
- (IBAction)deleteBtn:(id)sender
{
    // 删除第2、3行cell
    [self.weibosremoveObjectAtIndex:2];// 从数组中移除
    [self.weibosremoveObjectAtIndex:2];// 从数组中移除
    // 刷新表格
    //[self.tableView reloadData];
    // 带有的的动画的删除方式，需要指定参数
    [self.tableViewdeleteRowsAtIndexPaths:@[[NSIndexPathindexPathForItem:2inSection:0],
                                             [NSIndexPath indexPathForItem:3 inSection:0]
                                             ] withRowAnimation:UITableViewRowAnimationLeft];

}
```

- 结果如下：

![](../images/屏幕快照 2015-06-06 00.00.22.png)

- 3.5、总结
- 数据刷新的原则
   - 1、通过修改模型数据，来修改tableView的展示
   - 2、先修改模型数据
   - 3、再调用数据刷新方法
   - 4、不要直接修改cell上面子控件的属性

- 4、UITableView 编辑模式
   - 进入编辑模式需要设置一个属性editing。

- 4.1、进入编辑模式
   - 添加一个按钮，响应点击事件，改变UITableView的状态。默认是进入删除模式，还有一个添加模式。

![](../images/屏幕快照 2015-06-06 21.32.28.png)

```objc
// 编辑按钮
- (IBAction)editBtn:(id)sender
{
    // 获取状态，并取反
    [self.tableViewsetEditing:!self.tableView.isEditinganimated:YES];
}
```

- 进入添加模式需要实现代理方法：

```objc
/**

 * 这个方法决定了编辑模式时，每一行的编辑类型：insert（+按钮）、delete（-按钮）

 */

- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath

{

    returnUITableViewCellEditingStyleInsert;

}
```

- 4.2、左划删除cell
    - 实现一个代理方法对左划进行响应。

```objc
/**
 *  实现这个方法即可实现左划删除数据，并且可以响应添加和删除操作
 */
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 左划删除
    if (editingStyle == UITableViewCellEditingStyleDelete) // 删除模式
    {
        [self.weibos removeObjectAtIndex:indexPath.row];// 从数组中移除
        // 刷新表格
        //[self.tableView reloadData];
        // 带有的的动画的删除方式，需要指定参数
        [tableView deleteRowsAtIndexPaths:@[indexPath]withRowAnimation:UITableViewRowAnimationLeft];
    }
    elseif(editingStyle == UITableViewCellEditingStyleInsert) // 添加模式
    {
        NSLog(@"UITableViewCellEditingStyleInsert--");
    }
}
```

- 4.3、批量删除方法1

- 给xib添加一个图片控件，显示选中状态,用于表示多选的状态，选中后如下面所示状

![](../images/屏幕快照 2015-06-06 20.22.27.png)


- 然后给模型添加一个属性，用于表示cell的选中状态

```objc
/*是否选中*/
@property (assign, nonatomic,getter=isChecked) BOOL checked;
```
- 接着在封装cell类中得setter方法中对状态进行判断

```objc
// setter 方法
- (void)setWeibo:(SLQWeibo *)weibo
{
    _weibo = weibo;

    // 是否是选中状态
    self.checkImage.hidden = !weibo.isChecked;

    self.nameLable.text = weibo.name;
    self.iconImage.image = [UIImage imageNamed:weibo.icon];
    if (weibo.isVip)
    {
        self.vipImage.hidden = NO;
        self.vipImage.image = [UIImage imageNamed:@"vip"];
        self.nameLable.textColor = [UIColor redColor];
    }
    else
    {
          self.nameLable.textColor = [UIColor blackColor];
        self.vipImage.hidden = YES;
    }
    self.contentLable.text = weibo.text;

    if(weibo.picture)
    {
        self.pictureImage.hidden = NO;
        self.pictureImage.image = [UIImage imageNamed:weibo.picture];
    }
    else
    {
        self.pictureImage.hidden = YES;
    }

    // 计算cell高度v
    // 强制布局
    [selflayoutIfNeeded];

    // 计算高度
    if (self.pictureImage.hidden)
    {
         weibo.cellHeight = CGRectGetMaxY(self.contentLable.frame) + 10;
    }
    else
    {
         weibo.cellHeight = CGRectGetMaxY(self.pictureImage.frame) + 10;
    }
}
```

- 接着响应选中cell方法： didSelectRowAtIndexPath

```objc
/**
 *  cell 选中会调用这个方法
 */
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:YES]; // 取消选中后cell北京变灰的状态
    //取出模型数据进行修改
    SLQWeibo *weibo = self.weibos[indexPath.row]; // 取出选中的cell模型数据
    weibo.checked = !weibo.isChecked; // 显示或者隐藏选中图标
    [tableView reloadData]; // 刷新表格
}
```

- 最后响应删除按钮

```objc
// 删除按钮
- (IBAction)deleteBtn:(id)sender
{
    // 删除多行数据
    NSMutableArray *deleteArray = [NSMutableArray array];
    for (SLQWeibo *weibo in self.weibos)
    {
        if (weibo.isChecked)
        {
            [deleteArray addObject:weibo]; // 将选中的cell添加到待删除数组
        }
    }
    // 删除所有选中数据
    [self.weibos removeObjectsInArray:deleteArray];
    // 更新表格
    [self.tableView reloadData];
}
```

![](../images/屏幕快照 2015-06-06 21.29.31.png)

- 4.4、批量删除方法2
- 这里使用一个待删除数组记录选中的cell，然后点击删除按钮后删除选中所有选中cell.
- 首先，增加一个数组属性并初始化

```objc
// 待删除数组
@property (strong, nonatomic) NSMutableArray *deleteWeibo;

- (void)viewDidLoad {
    [superviewDidLoad];
    _deleteWeibo = [NSMutableArray array]; // 初始化数组
}
```

- 其次，在选中cell后进行判断 didSelectRowAtIndexPath

```objc
/**
 *  cell 选中会调用这个方法
 */
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:YES]; // 取消选中后cell北京变灰的状态
    //取出模型数据进行修改
    SLQWeibo *weibo = self.weibos[indexPath.row]; // 取出选中的cell模型数据
    if ([self.deleteWeibo containsObject:weibo]) // 如果包含在数组中则从数组中移除
    {
        [self.deleteWeibo removeObject:weibo];
    }
    else
    {
        [self.deleteWeibo addObject:weibo]; // 否则就添加到数组
    }

    [tableView reloadData]; // 刷新表格
}
```

- 再次，在刷新表格时在 cellForRowAtIndexPath 方法中判断cell状态后再显示

```objc
- (UITableViewCell *) tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 获取cell对象
    SLQWeiboCell *cell = [SLQWeiboCell cellWithTableView:tableView ];
    cell.weibo = self.weibos[indexPath.row];
    // 判断选中状态
    cell.checkImage.hidden = ![self.deleteWeibo containsObject:cell.weibo];
    return  cell;
}
```

- 最后，实现删除方法

```objc
// 删除按钮
- (IBAction)deleteBtn:(id)sender
{
    [self.weibos removeObjectsInArray:self.deleteWeibo];
    // 更新表格
    [self.tableView reloadData];

    // 清空删除数组
    [self.deleteWeibo removeAllObjects];
}
```

- 4.5、批量删除方法3：UITableView自带的方法
- UITableView 自带的批量删除操作，操作方式需要设置一个属性。
- 首先，在加载tableView时设置属性，这样就可以在进入编辑模式时支持多选操作。

```objc
- (void)viewDidLoad {
    [superviewDidLoad];
    // 允许在编辑模式下多选
    self.tableView.allowsMultipleSelectionDuringEditing = YES;
}
```

![](../images/屏幕快照 2015-06-06 22.48.20.png)

- 其次，就在就是在点击删除按钮后进行操作，因为选中的cell有tableView自己保存，直接获取就好了。

```objc
// 删除按钮
- (IBAction)deleteBtn:(id)sender
{
    NSArray *selected = [self.tableView indexPathsForSelectedRows];
    NSMutableArray *delete = [NSMutableArrayarray];
    for (NSIndexPath * obj in selected)
    {
        [delete addObject:self.weibos[obj.row]];
    }
    // 删除选中项
    [self.weibosremoveObjectsInArray:delete];
    // 更新表格
    [self.tableViewreloadData];
}
```
- 最后，其他的代码如下，大部分和上一个设置一样。

```objc
/**
 *  cell 选中会调用这个方法
 */
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    returnself.weibos.count;
}

- (UITableViewCell *) tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 获取cell对象
    SLQWeiboCell *cell = [SLQWeiboCell cellWithTableView:tableView ];
    cell.weibo = self.weibos[indexPath.row];
    return  cell;
}
```

![](../images/屏幕快照 2015-06-06 22.53.54.png)

- 5、tableView其他属性
    - tableFooterView / tableHeaderView
    - 头部和尾部显示控件，默认都是水平垂直居中，并填充
```objc
    // 给tableView 添加头部显示的View
    SLQPageScroll *pageView = [SLQPageScroll pageScroll];
    pageView.imageNames = @[@"ad_01",@"ad_02",@"ad_03",@"ad_04"];
    self.tableView.tableHeaderView = pageView;
    // 尾部添加控件
    SLQLoadData *loadData = [SLQLoadData loadData];
    // 指定控制器为代理对象
    loadData.delegate = self;
    self.tableView.tableFooterView = loadData;
```

- 6、自定义cell上的按钮

![](../images/屏幕快照 2015-06-14 10.00.47.png)

- 这是IOS8.0新增的一个代理方法

```objc
    // 自定义按钮样式
    - (NSArray *)tableView:(UITableView *)tableView editActionsForRowAtIndexPath:(NSIndexPath *)indexPath
    {
        // 创建按钮样式
        UITableViewRowAction *action1 = [UITableViewRowActionrowActionWithStyle:UITableViewRowActionStyleDefaulttitle:@"增加"handler:^(UITableViewRowAction *action, NSIndexPath *indexPath) {
            // 处理按钮按下事件
        }];

        action1.backgroundColor = [UIColor blueColor];
        UITableViewRowAction *action2 = [UITableViewRowActionrowActionWithStyle:UITableViewRowActionStyleDefaulttitle:@"删除"handler:^(UITableViewRowAction *action, NSIndexPath *indexPath) {
            // 删除模型数据
            [self.contacts removeObjectAtIndex:indexPath.row];
            // 刷新表格
            [self.tableView reloadData];
        }];
        UITableViewRowAction *action3 = [UITableViewRowActionrowActionWithStyle:UITableViewRowActionStyleDefaulttitle:@"haha"handler:^(UITableViewRowAction *action, NSIndexPath *indexPath) {

        }];
        return @[action1,action2,action3];
    }
```





http://www.cnblogs.com/songliquan/

