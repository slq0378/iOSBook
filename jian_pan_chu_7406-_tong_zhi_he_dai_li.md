# 键盘处理-通知和代理

## 通知和代理
- 处理文本输入框的输入事件，单击文本输入框后要弹出键盘。
- 弹出键盘有两种实现方式:一种代理，一种通知。也就是对应的（观察者模式和代理模式）。

## 1、通知

### 1.1、准备工作
- 每一个应用程序都有一个通知中心(`NSNotificationCenter`)实例，专门负责协助不同对象之间的消息通信。
- 任何一个对象都可以向通知中心发布通知(`NSNotification`)，描述自己在做什么。
- 其他感兴趣的对象(`Observer`)可以申请在某个特定通知发布时(或在某个特定的对象发布通知时)收到这个通知。

- 通知中心(`NSNotificationCenter`)提供了相应的方法来帮助发布通知

```objc
// 发布一个notification通知，可在notification对象中设置通知的名称、通知发布者、额外信息等
- (void)postNotification:(NSNotification *)notification;
// 发布一个名称为aName的通知，anObject为这个通知的发布者
- (void)postNotificationName:(NSString *)aName object:(id)anObject;

// 发布一个名称为aName的通知，anObject为这个通知的发布者，aUserInfo为额外信息
- (void)postNotificationName:(NSString *)aName object:(id)anObject` userInfo:(NSDictionary *)aUserInfo;
```

- 通知中心(`NSNotificationCenter`)提供了方法来注册一个监听通知的监听器(Observer)

```objc
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString *)aName object:(id)anObject;
    observer：监听器，即谁要接收这个通知
    aSelector：收到通知后，回调监听器的这个方法，并且把通知对象当做参数传入
    aName：通知的名称。如果为nil，那么无论通知的名称是什么，监听器都能收到这个通知
    anObject：通知发布者。如果为anObject和aName都为nil，监听器都收到所有的通知

- (id)addObserverForName:(NSString *)name object:(id)obj queue:(NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block;
    name：通知的名称
    obj：通知发布者
    block：收到对应的通知时，会回调这个block
    queue：决定了block在哪个操作队列中执行，如果传nil，默认在当前操作队列中同步执行
```
- 通知中心不会保留(retain)监听器对象，在通知中心注册过的对象，必须在该对象释放前取消注册。否则，当相应的通知再次出现时，通知中心仍然会向该监听器发送消息。因为相应的监听器对象已经被释放了，所以可能会导致应用崩溃。

- 通知中心提供了相应的方法来取消注册监听器

```objc
- (void)removeObserver:(id)observer;
- (void)removeObserver:(id)observer name:(NSString *)aName object:(id)anObject;
```

- 实现过程：先注册消息 postNotificationName ，然后在响应的方法的接收消息 addObserver，最后实现接收消息的方法

- 因为在系统中又定义好的键盘处理通知,所以直接进行监听就行了。

```objc
UIKIT_EXTERN NSString *const UIKeyboardWillShowNotification; // 即将弹出
UIKIT_EXTERN NSString *const UIKeyboardDidShowNotification; // 显示
UIKIT_EXTERN NSString *const UIKeyboardWillHideNotification;   //即将隐藏
UIKIT_EXTERN NSString *const UIKeyboardDidHideNotification; // 隐藏
```

### 1.2、实现过程

```objc
//  1、通过通知监听键盘出现消息
// 添加监听器，监听键盘弹出
[[NSNotificationCenter defaultCenter] addObserver:selfselector:@selector(keyboardComeout:) name:UIKeyboardWillShowNotificationobject:nil];
// 添加监听器,监听键盘退出
[[NSNotificationCenter defaultCenter] addObserver:selfselector:@selector(keyboardGoaway:) name:UIKeyboardWillHideNotificationobject:nil];

// 2、响应通知的方法，显示键盘
- (void)keyboardComeout:(NSNotification *)note
{
    NSLog(@"keyboardComeout--\n%@",note);
    // 取出消息中键盘出现后的最终位置,要将字典对象转换成CGRect
    CGRect rect = [note.userInfo[UIKeyboardFrameEndUserInfoKey] CGRectValue];
    self.viewBotom.constant = rect.size.height;
    // 取出键盘弹出所需要的时间
    double animateTime = [note.userInfo[UIKeyboardAnimationDurationUserInfoKey] doubleValue];
    [UIViewanimateWithDuration:animateTime animations:^{
        [self.viewlayoutIfNeeded];
    }];
    /**
     *   NSNotification 内部信息
     NSConcreteNotification 0x7f896bebf4d0 {name = UIKeyboardWillShowNotification; userInfo = {
     UIKeyboardAnimationCurveUserInfoKey = 7;
     UIKeyboardAnimationDurationUserInfoKey = "0.25";
     UIKeyboardBoundsUserInfoKey = "NSRect: {{0, 0}, {375, 225}}";
     UIKeyboardCenterBeginUserInfoKey = "NSPoint: {187.5, 779.5}";
     UIKeyboardCenterEndUserInfoKey = "NSPoint: {187.5, 554.5}";
     UIKeyboardFrameBeginUserInfoKey = "NSRect: {{0, 667}, {375, 225}}";
     UIKeyboardFrameEndUserInfoKey = "NSRect: {{0, 442}, {375, 225}}";
     }}

     */
}
/**
 *  3、开始滚动界面后隐藏键盘
 */
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView
{
    //退出键盘
    // NSLog(@"scrollViewWillBeginDragging--");
    //[self.view endEditing:YES];//方法1
    [self.textField resignFirstResponder]; // 方法2：谁响应的键盘，谁就可以调用这个方法退出键盘(resignFirstResponder)
}
// 隐藏键盘
- (void)keyboardGoaway:(NSNotification *)note
{
    NSLog(@"keyboardGoaway--\n%@",note);
    self.viewBotom.constant = 0;
     //取出键盘弹出所需要的时间
    double animateTime = [note.userInfo[UIKeyboardAnimationDurationUserInfoKey] doubleValue];
    [UIViewanimateWithDuration:animateTime animations:^{
        [self.viewlayoutIfNeeded];
    }];
}
```

- 看弹出效果，实际效果是有动画的，过渡更好。

Test



### 1.3、键盘弹出后还要对真个View进行上移，隐藏后下移

- 这里使用更新控件的约束来实现view的移动。

屏幕快照 2015 06 07 15 37 51

- 如下属性：

```objc
@property (weak, nonatomic) IBOutletNSLayoutConstraint *viewBotom;
```

- 然后控制约束的constant值，就可以实现view的移动。

- 设置方法方法：

```objc
// 取出消息中键盘出现后的最终位置,要将字典对象转换成CGRect

CGRect rect = [note.userInfo[UIKeyboardFrameEndUserInfoKey] CGRectValue];

self.viewBotom.constant = rect.size.height;
```

### 1.4、给UITextField 两边添加一个间距

- 直接使用其自带的功能即可

```objc
// 设置文本框左边的内容
UIView *leftView = [[UIView alloc] init];
leftView.frame = CGRectMake(0, 0, 10, 0);
self.messageField.leftView = leftView;
self.messageField.leftViewMode = UITextFieldViewModeAlways;
// 右边间距
self.messageField.rightView = leftView;
self.messageField.rightViewMode = UITextFieldViewModeAlways;
```

### 1.5、使用完毕后要手动删除监听器

```objc
- (void)dealloc
{
    // 使用完毕后要手动移除监听对象
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

## 2、代理模式

### 2.1、使用注意事项

- 1、在头文件中实现代理，定义协议
- 2、使用方法：1 设置代理，2 遵守协议，3 实现方法
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
    - 5、 yyy对象遵守协议，实现代理方法

### 2.2、代码实现

- 先看效果

    屏幕快照 2015 06 08 15 16 58

- 首先在xib里实现界面如下：

    屏幕快照 2015 06 08 15 13 14

- 然后实现文件也比较简单，在点击过加载按钮后，由此对象给代理发送加载数据通知。
    - 头文件实现里包括代理所遵守协议的定义，包括一个可用方法。

```objc
@classSLQLoadData;

@protocol SLQLoadDataDelegate <NSObject>
@optional

/**
 *  代理方法：点击按钮操作
*/
- (void)LoadDataDidCilckButton:(SLQLoadData *)loadData;

@interface SLQLoadData : UIView
// 代理属性
@property (weak,nonatomic) id<SLQLoadDataDelegate> delegate;

+ (instancetype)loadData;// 返回加载对象
- (void)endingloadData; // 结束加载
@end

#import "SLQLoadData.h"
@interfaceSLQLoadData ()
- (IBAction)loadMoreData:(id)sender;
@property (weak, nonatomic) IBOutletUIView *loadView; // 正在加载界面
@property (weak, nonatomic) IBOutletUIButton *loadBtn; // 加载按钮
@end

@implementation SLQLoadData
+ (instancetype)loadData
{
    // NSStringFromClass 将类名转换成字符串,xib文件名和类名一样
    return [[[NSBundlemainBundle] loadNibNamed:NSStringFromClass(self) owner:nil options:nil] lastObject];
}
- (IBAction)loadMoreData:(id)sender
{
    // 模拟从网络加载数据
    //NSLog(@“loadMoreData”);
    self.loadView.hidden = NO;
    self.loadBtn.hidden = YES;
    // 发送消息到代理对象,首先判断代理方法是否存在
    if ([self.delegate respondsToSelector:@selector(LoadDataDidCilckButton:)])
    {
        [self.delegate  LoadDataDidCilckButton:self];
    }
}
// 隐藏正在加载界面
- (void)endingloadData
{
    self.loadView.hidden = YES;
    self.loadBtn.hidden = NO;
}
@end
```

### 2.3、控制器中对代理方法的使用

- 1、包含头文件并遵守协议

```objc
#import "SLQLoadData.h"

@interfaceViewController () <SLQLoadDataDelegate> // 遵守代理协议
        2、指定代理对象

    SLQLoadData *loadData = [SLQLoadData loadData];
    // 指定控制器为代理对象
    loadData.delegate = self;
    self.tableView.tableFooterView = loadData;
       3、实现代理方法

// 实现代理方法
- (void)LoadDataDidCilckButton:(SLQLoadData *)loadData
{
    NSLog(@"LoadDataDidCilckButton");
    // 模拟从网络加载数据,加一个定时器
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{

        SLQFengNews *news = [[SLQFengNews alloc] init];
        news.title = @"高考往事";
        news.topic = NO;
        news.image = @"ad_02";
        news.comment = @"13489";
        // 添加至模型对象
        [self.news addObject:news];
        // 刷新表格
        [self.tableView reloadData];

        // 隐藏loadView
        SLQLoadData *load = (SLQLoadData *)self.tableView.tableFooterView;
        [load endingloadData];
    });

}
```

### 2.4、最终效果 -_-
- LoadData



## 总结

- 其他系统自带通知

- UIDevice类提供了一个单粒对象，它代表着设备，通过它可以获得一些设备相关的信息，比如电池电量值(batteryLevel)、电池状态(batteryState)、设备的类型(model，比如iPod、iPhone等)、设备的系统(systemVersion)

- 通过[UIDevice currentDevice]可以获取这个单例对象

```objc
// UIDevice对象会不间断地发布一些通知，下列是UIDevice对象所发布通知的名称常量：
UIDeviceOrientationDidChangeNotification // 设备旋转
UIDeviceBatteryStateDidChangeNotification // 电池状态改变
UIDeviceBatteryLevelDidChangeNotification // 电池电量改变
UIDeviceProximityStateDidChangeNotification // 近距离传感器(比如设备贴近了使用者的脸部)
```

- 键盘通知
    - 我们经常需要在键盘弹出或者隐藏的时候做一些特定的操作,因此需要监听键盘的状态

```objc
// 键盘状态改变的时候,系统会发出一些特定的通知
UIKeyboardWillShowNotification // 键盘即将显示
UIKeyboardDidShowNotification // 键盘显示完毕
UIKeyboardWillHideNotification // 键盘即将隐藏
UIKeyboardDidHideNotification // 键盘隐藏完毕
UIKeyboardWillChangeFrameNotification // 键盘的位置尺寸即将发生改变
UIKeyboardDidChangeFrameNotification // 键盘的位置尺寸改变完毕

// 系统发出键盘通知时,会附带一下跟键盘有关的额外信息(字典),字典常见的key如下:
UIKeyboardFrameBeginUserInfoKey // 键盘刚开始的frame
UIKeyboardFrameEndUserInfoKey // 键盘最终的frame(动画执行完毕后)
UIKeyboardAnimationDurationUserInfoKey // 键盘动画的时间
UIKeyboardAnimationCurveUserInfoKey // 键盘动画的执行节奏(快慢)
```

- 代理和通知比较
- 共同点
    - 利用通知和代理都能完成对象之间的通信(比如A对象告诉D对象发生了什么事情, A对象传递数据给D对象)
- 不同点
    - 代理 : 1个对象只能告诉另1个对象发生了什么事情
    - 通知 : 1个对象能告诉N个对象发生了什么事情, 1个对象能得知N个对象发生了什么事


