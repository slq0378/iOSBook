# QQ聊天布局实现总结

## 分析
- 1、按钮内部边距 contentInset，高度需要重新设置约束
- 2、UITextFiled  leftView,空出左边一点间距
- 3、整个View往上移动，修改view的底部约束
- 4、监听键盘通知事件NSNotificationCenter
	- 1、发送事件到通知中心呢 postNotification
	- 2、注册监听器addObserver，接收消息observeNotification
	- 3、使用完毕后dealloc，要移除监听对象，removeObserver
- 5、系统自带的通知：
	- 1、UIDevice-单例对象 设备状态：旋转，电量等
	- 2、键盘通知UIKeyboard
- 6、键盘的弹出和隐藏
	- 1、注册对象，接收弹出消息，显示键盘，修改约束
	- 2、隐藏键盘 修改view的属性，endEditing，修改约束
	- 3、在对象摧毁时要移除监听对象removeObserver
	- 4、其他方法：UIKeyboardWillChangeFrameNotifation（监听frame状态）
- 7、transform属性
	- 1、平移：CGAffineTransformMakeTranslation
	- 2、缩放:  CGAffineTransformMakeScale
	- 3、旋转：CGAffineTransformMakeRotation 弧度值
	- 4、叠加操作:边平移变缩放：CGAffineTransformScale
	- 5、清零： CGAffineTransformIdentify，所有改变恢复原状
- 8、使用transform改变键盘位置
- 9、程序加载图标
    - AppIcon ：程序图标
    - lanchImage ；运行前加载的图片
- 10、UIButton 内部结构分析（UIImage+UILable）
	- 1、设置按钮的图片或者文字一定要指定状态，不然无法显示
	- 2、修改textLable、图片等子控件的位置：imageRectForContentRect/titleRectForContentRect或者使用layoutSubViews一起进行设置
- 11、头部尾部
	- 1、tableFooterView、tableHeaderView,默认宽度填充整个cell，居中显示
	- 2、可以在头部或者尾部添加自定义的view，显示需要显示的各种控件（比如尾部退出按钮，头部滚动View）
	- 3、示例：尾部加载数据（）
	- 4、监听按钮响应：代理模式
- 12、代理模式
	- 1、在头文件中实现代理，定义协议
	- 2、使用方法：设置代理，遵守协议，实现方法
	- 3、规范：参考苹果官方的协议实现方式
	- 4、作用:
		- 1、监听事件：A是监听B的一些行为，则A成为B的代理
		- 2、消息传递：A想告诉B一些事情，则B成为A的代理
	- 5、总结：
		- 1、如果想监听别人的一些行为，那么就要成为别人的代理
		- 2、如果你想告诉别人一些事情，那么就让别人成为你得代理
	- 6、开发步骤
		- 1、拟一份协议（格式：控件名 + Delegate），在协议里声明一些代理方法（一般代理方法都是@optional）
		- 2、声明一个代理属性（弱引用） @property (weak,nonatomic) id delegate;
		- 3、在内部发生某些行为时，调用代理对应的代理方法，通知代理内部发生了什么事情
		- 4、设置代理： xxx.delegate = self;
		- 5、yyy对象遵守协议，实现代理方法
- 13、代理和通知的区别
	- 1、代理：一个对象只能告诉另外一个对象( 1对1 )
	- 2、通知：多个通知面向N个对象（多对多）
- 14、KVO (监听属性)
	KVC ：key value coding （给模型属性赋值）
	KVO ：key value observing  (监听对象的某个属性值的改变)
	- 1、给属性添加监听：addObserve: forKeyPath:
	- 2、监听方法是系统自带的方法：observeValueForKeyPath: : ：
	- 3、同样在dealloc中删除：removeObserve：：


## 代理
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


``` objc
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

## QQ聊天界面实现

- 效果图

![](../images/屏幕快照 2015-06-17 14.12.30.png)

### 实现过程：

- 1、首先实现基本界面
    - 头像使用 UIImageView ：
    - 文字消息使用 UIButton
    - 标签使用 UILable ：水平居中
    - 所有元素在一个cell中，在加载cell时进行判断显示和隐藏。
    - 合理设置各个控件之间的约束关系。主要是UIIimageVIew和UIButton顶部对齐，间距为10。UIButton的宽度设置一个约束范围,比如说 (>=60 &&  <=300);
    - 底部添加一个UIView ，添加输入框等。

- 2、创建模型文件
    - 所有元素在一个cell中，在加载cell时进行判断显示和隐藏。
    - 按照message.plist文件内容添加需要的属性，然后添加一个cellHeight属性计算cell高度，和一个决定是否显示时间到cell得属性hideTime。

```objc
#import <UIKit/UIKit.h>

// 枚举类型，
typedefenum{
    SLQMessageTypeMe = 0,
    SLQMessageTypeOther = 1
}SLQMessageType;

@interface SLQMessage : NSObject
/*内容*/
@property (strong, nonatomic) NSString *text;
/*时间*/
@property (strong, nonatomic) NSString *time;
/*类型*/
@property (assign, nonatomic) SLQMessageType type;

/*cellHeight*/
@property (assign, nonatomic) CGFloat cellHeight;
/*是否隐藏时间*/
@property (assign, nonatomic,getter=isHideTime) BOOL hideTime;

+ (instancetype)MessageWithDict:(NSDictionary *)dict;

@end
```

- 实现文件

```objc
#import "SLQMessage.h"

@implementation SLQMessage

+(instancetype)MessageWithDict:(NSDictionary *)dict
{
    SLQMessage *message = [[SLQMessage alloc] init];
    [message setValuesForKeysWithDictionary:dict];
    return  message;
}

@end
```
- 这里需要注意的就是枚举类型的使用，如果在一个类中要定义枚举类型，那么命名规则就是：
- 以类名开头后面直接跟操作标识；如 SLQMessage + Type；

- 3、实现对cell操作的封装

```objc
#import <UIKit/UIKit.h>
@classSLQMessage;
@interface SLQMessageCell : UITableViewCell

/*模型对象*/
@property (strong, nonatomic) SLQMessage *message;

+ (instancetype)cellWithTableView:(UITableView *)tableView;

@end
```

- 对tableView的每一个控件拖线建立关联。然后重写setter方法，对控件进行赋值。

```objc
#import "SLQMessageCell.h"
#import "SLQMessage.h"

//define this constant if you want to use Masonry without the 'mas_' prefix
#define MAS_SHORTHAND
//define this constant if you want to enable auto-boxing for default syntax
#define MAS_SHORTHAND_GLOBALS
#import "Masonry.h"


@interfaceSLQMessageCell ()
@property (weak, nonatomic) IBOutletUILabel *timeLable;
@property (weak, nonatomic) IBOutletUIButton *meBtn;
@property (weak, nonatomic) IBOutletUIImageView *meImage;
@property (weak, nonatomic) IBOutletUIButton *otherBtn;
@property (weak, nonatomic) IBOutletUIImageView *otherImage;
@end

@implementation SLQMessageCell

// 重写setter方法

- (void)setMessage:(SLQMessage *)message
{
    _message = message;

    self.backgroundColor = [UIColorbrownColor];
    if(message.isHideTime) // 隐藏时间
    {
        self.timeLable.hidden = YES;
        [self.timeLableupdateConstraints:^(MASConstraintMaker *make) {
            make.height.equalTo(0); // 高度为0
        }];
    }
    else
    {
        self.timeLable.text = message.time;
        self.timeLable.hidden = NO;
        [self.timeLableupdateConstraints:^(MASConstraintMaker *make) {
            make.height.equalTo(22);
        }];
    }

    if (message.type == SLQMessageTypeMe)
    {
        [selfsetShowBtn:self.meBtnWithShowImage:self.meImageWithHideBtn:self.otherBtnWithHideImage:self.otherImage];
    }
    if (message.type == SLQMessageTypeOther)
    {
        [selfsetShowBtn:self.otherBtnWithShowImage:self.otherImageWithHideBtn:self.meBtnWithHideImage:self.meImage];
    }
}
```
- 因为每次显示cell都要进行计算，将cell的显示封装到方法中。

```objc
// 显示隐藏控件并计算控件的高度
- (void)setShowBtn:(UIButton *)showBtn WithShowImage:(UIImageView *)showImage WithHideBtn:(UIButton *)hideBtn WithHideImage:(UIImageView *)hideImage
{
    [showBtn setTitle:self.message.textforState:UIControlStateNormal];

    // 隐藏其他
    hideBtn.hidden = YES;
    hideImage.hidden = YES;
    // 显示自己
    showBtn.hidden = NO;
    showImage.hidden = NO;

    // 强制更新
    [selflayoutIfNeeded];
    // 更新约束，设置按钮的高度就是textLable的高度
    [showBtn updateConstraints:^(MASConstraintMaker *make) {
        CGFloat buttonH = showBtn.titleLabel.frame.size.height;//
        make.height.equalTo(buttonH);
    }];
    // 强制更新
    [selflayoutIfNeeded];
    CGFloat btnMaxY = CGRectGetMaxY(showBtn.frame);
    CGFloat imageMaxY = CGRectGetMaxY(showImage.frame);
    // 设置cell高度
    self.message.cellHeight = MAX(btnMaxY, imageMaxY) + 10;
}
```
- 其他方法和以往一样

```objc
+ (instancetype)cellWithTableView:(UITableView *)tableView
{
    SLQMessageCell *cell = [tableView dequeueReusableCellWithIdentifier:@"message"];
    return cell;
}
- (void)awakeFromNib {
    // Initialization code
    // 多行显示
     self.meBtn.titleLabel.numberOfLines=0;
     self.otherBtn.titleLabel.numberOfLines=0;
}
```

- 4、接下来说说按钮背景的问题
    - 按钮背景默认填充整个按钮，但是默认情况下的填充效果不是很好。

- 如下代码：

```objc
    UIImageView *imageView = [[UIImageView alloc] init];
    imageView.frame = CGRectMake(10, 10, 300, 200);
    UIImage *image = [UIImage imageNamed:@"chat_send_nor"];

    // 方法1 ,设置拉伸间距，默认拉伸中心1*1像素
    //image = [image stretchableImageWithLeftCapWidth:image.size.width * 0.5 topCapHeight:image.size.height * 0.5];
    // 方法2 设置边界
    UIEdgeInsets edge = UIEdgeInsetsMake(50, 40, 40, 40);
    //image = [image resizableImageWithCapInsets:edge ];

    // UIImageResizingModeStretch 拉伸模式
    // UIImageResizingModeTile 填充模式
    image = [image resizableImageWithCapInsets:edge resizingMode:UIImageResizingModeStretch];
    // 方法3
    // 在../images.xcassets中对图片进行设置

    imageView.image = image;
    [self.view addSubview:imageView];

    // 对比图片
    UIImageView *imageView1 = [[UIImageView alloc] init];
    imageView1.frame = CGRectMake(10, 210, 300, 200);
    UIImage *image1 = [UIImage imageNamed:@"chat_send_nor"];
    imageView1.image = image1;


    [self.view addSubview:imageView1];
```

- 会出现以下效果，默认是下边的图片，所以有必要对图片进行拉伸。

![](../images/屏幕快照 2015-06-07 16.21.42.png)

- 其中方法3的设置是将图片导入Image.xcassets中后选中图片设置。

![](../images/屏幕快照 2015-06-06 14.30.58.png)

- 可以通过代码设置按钮的内间距

```objc
    // 可以这样设置内间距
    UIEdgeInsets edge = UIEdgeInsetsMake(15, 15, 15, 15);
    [showBtn setTitleEdgeInsets:edge];
```
- 或者直接在按钮的属性里设置

![](../images/屏幕快照 2015-06-07 16.08.34.png)

- 设置过间距后，就可以计算btn的高度时，因为textlable的高度不固定，所以让btn的高度等于textLable 的高度。但是又因为按钮背景图片的边缘有一部分是透明的，如下：红色是按钮，蓝色是图片。

![](../images/屏幕快照 2015-06-07 16.04.27.png)


- 所以显示文字高度会，这里对其按钮高度 + 30，而textLable默认会水平垂直居中。

![](../images/屏幕快照 2015-06-07 16.07.07.png)

- 5、在控制器中得实现方法和以往的一样
    - 只需要在这里判断以下消息显示的时间是否一致，如果一致就隐藏。

```objc
- (NSMutableArray *)messages
{
    if (_messages == nil)
    {
        NSArray *dictArray = [NSArrayarrayWithContentsOfFile:[[NSBundlemainBundle] pathForResource:@"messages.plist"ofType:nil]];
        NSMutableArray *tempArray = [NSMutableArrayarray];
        // 记录上一个message，判断是否显示时间
        SLQMessage *lastMessage = nil;
        for (NSDictionary *dict in dictArray)
        {
            SLQMessage *message = [SLQMessage MessageWithDict:dict];
            message.hideTime = [message.time isEqualToString:lastMessage.time];
            [tempArray addObject:message];
            // 重新赋值
            lastMessage = message;
        }
        _messages = tempArray;
    }
    return_messages;


}

- (void)viewDidLoad {
    [superviewDidLoad];
}
/**
 *  tableView 行数
 */
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    //NSLog(@"%zd",self.messages.count);
    returnself.messages.count;
}
/**
 *  设置每一个cell
 */
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    SLQMessageCell *cell = [SLQMessageCellcellWithTableView:tableView];

    cell.message = self.messages[indexPath.row];

    return  cell;
}
/**
*  设置cell高度
*/
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    SLQMessage *message = self.messages[indexPath.row];
    return message.cellHeight;
}
/**
 *  给出预估高度
 */
- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    return  200;
}

@end
```

- 总结：
    - 这是一种方法，还有其他的实现方法，接下来尝试一下。

- 5、用两个cell实现界面

![](../images/屏幕快照 2015-06-07 17.32.08.png)

- 只需改动一些代码就行。
    - 1、改动每个cell的标志 一个是me，一个是other
    - 2、修改setter方法

```objc
// 重写setter方法
- (void)setMessage:(SLQMessage *)message
{
    _message = message;

    self.backgroundColor = [UIColorbrownColor];
    if(message.isHideTime) // 隐藏时间
    {
        self.timeLable.hidden = YES;
        [self.timeLableupdateConstraints:^(MASConstraintMaker *make) {
            make.height.equalTo(0);
        }];
    }
    else
    {
        self.timeLable.text = message.time; // 显示时间
        self.timeLable.hidden = NO;
        [self.timeLableupdateConstraints:^(MASConstraintMaker *make) {
            make.height.equalTo(22);
        }];
    }
    //
    [self.contentBtnsetTitle:message.textforState:UIControlStateNormal];
    // 强制布局
    [selflayoutIfNeeded];
    // 添加约束
    [self.contentBtnupdateConstraints:^(MASConstraintMaker *make) {
        CGFloat textLableHeight = self.contentBtn.titleLabel.frame.size.height + 30;
        make.height.equalTo(textLableHeight);
    }];

    [selflayoutIfNeeded];
    CGFloat btnMaxY = CGRectGetMaxY(self.contentBtn.frame);
    CGFloat iconMaxY = CGRectGetMaxY(self.iconImage.frame);
    message.cellHeight = MAX(btnMaxY, iconMaxY);
}
```
- 3、修改返回cell对象的方法，传入一个message用来判断是哪个cell

```objc
/**
 *  返回cell对象
 */
+ (instancetype)cellWithTableView:(UITableView *)tableView andMessage:(SLQMessage *)message
{
    NSString *ID =  (message.type == SLQMessageTypeMe)?@"me":@"other";
    SLQMessageCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];
    return cell;
}
```
- 4、在控制器中设置如下

```objc
/**
 *  设置每一个cell
 */
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 获取一个cell，根据类型
    SLQMessageCell *cell = [SLQMessageCell cellWithTableView:tableView andMessage:self.messages[indexPath.row]];

    cell.message = self.messages[indexPath.row];
    return  cell;
}
```

- 好了，效果一样。


### 更新: 滚动到最新的一行

- 响应一个发送按钮，点击发送按钮后获取文本框数据，并显示到tableView中。
- 滚动到最新一行
    - 发送按钮如下

```objc
/**
 *  发送信息
 */
- (IBAction)sendMessage:(id)sender
{
    // 获取文字内容
    NSString *message = self.textField.text;
    SLQMessageType type = (arc4random_uniform(2));
    // 更新模型数据
    SLQMessage *mess = [[SLQMessage alloc] init];
    mess.time = @"2015-11-23";
    mess.text = message;
    mess.type = type;
    // 设置模型数据,添加到数组
    [self.messages addObject:mess];

    // 刷新表格
    [self.tableViewreloadData];
    // 滚动到底部方法
    [selfscrollToBottom];
}
```

- 滚动底部方法

```objc
// 滚动到底部
- (void)scrollToBottom
{
    CGFloat yOffset = 0;
    // 如果tableView的高度大于tableView的自有有高度，y轴偏移量就等于contentSize - bounds
    if (self.tableView.contentSize.height > self.tableView.bounds.size.height) {
        yOffset = self.tableView.contentSize.height - self.tableView.bounds.size.height;
    }
    // 设置偏移量为最底部
    [self.tableViewsetContentOffset:CGPointMake(0, yOffset) animated:NO];
}
```
http://www.cnblogs.com/songliquan/
