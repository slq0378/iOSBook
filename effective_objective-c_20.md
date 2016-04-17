# Effective OC 2.0 学习笔记

- 1、OC的特殊之处
	- 函数调用语言的代码执行由编译器决定，如果是多态的话，则通过*虚函数表*在运行时再决定调用那个函数。
	- 消息结构语言函数的调用只会在运行时才会寻找要调用的函数。
- 2、将引入头文件的时机尽量后延，只有在明确需要时才引入。
    - 减少编译时间，头文件是在编译器引入的
    - 如果是协议的话，最好单独放在一个头文件 
    - 每次在头文件里引用其他头文件时都要问问自己是否可以用提前应用声明`@class`替换`#import`操作
- 3、多用字面量语法，少用与之等价的方法
	- 所谓字面量语法：`NSNumber *intNumber = @(1);`
	- 字面量数组：	`NSArray *animals = @[@"cat",@"dog",@"mouse"];`
	- 取值的话直接使用索引来实现： `NSString *dog = animals[1];`
	- 使用字面语法量更为安全，可以在出现nil值时给出提示
	- 字面量字典(key-value)：`NSDictionary *dict = @{@"first":@"001",@"second":@"002"};`
	- 使用字面量创建数组和字典时如果出现nil值会直接报错，所以也是一种很好的数据安全措施。
- 4、多用类型常量，少用#define预处理宏
	- `#deine AnimationDuration 3.0`
	- `static const NSTimeInterval kAnimationDuraton = 3.0;` 
	- 若长量在内部可见，则以k开头，若在所有类中可见，则以类名为前缀
	- 使用const 常量可以利用一些编译器特性
	- 如果准备在外部也使用这个常量，需要这样定义

	```objc
	//.h
	extern const NSTimeInterval ClassName+AnimationDuraton;
	//.m
	const NSTimeInterval ClassName+AnimationDuraton = 3.0;
	```
	
- 5、用枚举表示状态、选项、状态码 
    - 普通枚举 NS_ENUM , NS_OPTIONS
    - 处理枚举类型的switch语句不要出现default语句
    - 按位与 枚举 
- 6、属性 理解
    - 如果自己实现setter、getter方法，那么要在源文件中声明 `@dynamic 属性名`
- 7、在对象内部尽量直接访问实例变量
	- 点语法访问：经过getter方法
	- 直接访问：不经过getter方法，速度更快，不会触发KVO
	- **解决方法**：读取数据，直接访问，写入数据，属性（点语法）访问，在初始化方法和dealloc中，始终使用直接访问方式。
- 8、理解“对象等同性”这一概念
	- 若两个对象相等，那么hash码也相等
	- `==` 指针比较
	- `isEqual` - 对象
	- `isEqualToString` - 字符串
	- `hash`
- 9、以“类族模式”隐藏实现细节
    - 以抽象基类隐藏子类实现细节 
- 10、在既有类中使用关联对象存放自定义数据
	- 1、可以通过“关联机制”将两个对象关联起来
	- 2、关联时要指名拥有关系
	- 3、尽量避免使用关联，因为出现bug，很难调试
- 11、理解`objc_msgSend`的作用
	- 消息发送机制 
	- 查找过程：接收者类缓存->类方法列表->父类方法->找不到就进行消息转发
- 12、理解消息转发机制
	- 如果对象接收到未知的方法，会把这个方法转交给备用接收者，会调用方法`forwaringTargetorSelector:`如果这个方法还不能处理，就会进行完整的消息转发。
	- 消息转发机制分为两个步骤
	   - 第一、是在本类进行动态方法解析，会调用`+(BOOL)resolveInstanceMethod:(SEL)selector`或`+(BOOL)resolveClassMethod:(SEL)selector` ，其中`selector`就是那个位置的方法，可以在这里加入自己的处理逻辑，防止程序崩溃
	   - 第二、本类还有一次机会自己处理这个未知的方法，第一步没有效果的话，运行时会询问类是否进行消息转发`-(id)forwaringTargetorSelector:(SEL)selector`，`selector`表示未知的选择子，如果找到就返回新的接收对象，否则返回nil
	   - 第三，开启完整的消息转发，首先创建`NSInvocation`对象，把`selector`相关的信息包括接收者，方法名，和参数。这一步会调用`-(void)forwardInvocation:(NSInvocation) invocation `方法，然后逐层寻找，直到`NSObject`，如果还是没有，就调用`doesNotRecognizeSelector:`并抛出异常。
	- 可以在编写自己的类时，在消息转发过程中挂钩，用于执行预定的逻辑，而不是让程序崩溃掉。
- 13、方法交换技术
    - 在运行时动态交换方法的实现
    - `metho_exchangeImplementations(Method m1,Method m2)`
- 14、类对象
    - 每个对象都有一个Class类型的指针，用来表明其类型。
    - `isMemberOfClass:`和 `isKindOfClass:` 动态判断对象类型
- 15、用前缀避免命名冲突
    - 最好三个字母以上，苹果声称保留使用所有两个字母的前缀
    - 类，分类都要加前缀
    - 所有自定义的方法最好都加上前缀
- 16、“全能初始化方法”
    - 基础初始化方法，其他所有类似初始化方法都要调用这个方法。   
- 17、实现description方法
    - 给自定义类实现这个方法可以在调试时方便的看到各种信息。

```objc
- (NSString *)description {
    return [NSString stringWithFormat:@"%@ %@,_age,_name];
  }
```

- 18、尽量使用不可变对象
    - 对外暴露的属性，尽量使用不可变对象。
- 19、使用清晰而协调的命名方式
    - 名称不怕长，但一定要见名知意
- 20、为私有方法添加前缀 
    - 方便调试，可以很快的把私有方法和公共方法区分开
    - `- (void)slq_addNum;`
- 21、理解OC错误模型
    - `NSError`的使用
    - `-(BOOL)doSomething:(NSError**)error`
- 22、实现 `NSCopying` 协议
    - `NSCopying` 默认是浅拷贝，如果想实现深拷贝，自己实现方法
- 23、通过委托和数据源协议进行对象间通信
- 24、将类的实现代码分散到便于管理的数个分类之中
    - 如果类很庞大，那么可以将方法写到分类中，按模块区分
    - 将应该私有的方法归入‘private’分类中，以实现隐藏细节
- 25、总是为第三方类的分类添加名称前缀
    - 分类中方法也要添加相应前缀 
- 26、勿在分类中声明属性
    - 在分类中声明属性并不能实现属性相关的实例变量 
    - 如果一定要属性的存取方法，使用关键字 `@dynamic`
- 27、使用“class-cintinuation”分类隐藏实现细节
    - `class-cintinuation分类`是定义在实现文件中，对外隐藏实现
    - 这种方式我可以称为匿名分类，在匿名分类中可以定义方法和变量
    - 如果想隐藏某个方法或实例变量，可以把她定义在匿名分类中

```objc
@interface Person() 

@end
```

- 28、通过协议提供匿名对象
    - 协议对应的只是遵守协议的id类型对象
- 29、引用计数
- 30、ARC-自动引用计数
    - ARC规则：若方法名已一下词语开头，则返回的对象归调用者所有
        -  alloc
        -  new
        -  copy
        -  mutableCopy
    - ARC:显示保留新值，再释放旧值，最后设置实例变量。
- 31、在dealloc中只释放引用并解除监听
- 32、编写“异常安全代码”时留意内存管理问题
    - 除非必要，否则不要再OC中使用异常
- 33、循环引用
    - 使用weak避免循环
- 34、以自动释放池降低内存峰值
    - autoreleasepool  
- 35、使用 ”僵尸对象“ 调试内存问题
- 36、绝对不要使用retainCount
- 37、block
    - `return_type (^block_name)(parameters)`
    - 注意循环引用
- 38、为常用的块类型创建`typedef`
- 39、使用handler块降低代码分散程度
- 40、使用block时注意循环引用
- 41、多用GCD进行线程同步，而不是同步锁
- 42、多用GCD,少用performSelector系列方法
- 43、GCD和NSOperation的区别
    -  NSOperation的高级功能
        -  取消某个操作
        -  制定操作间的依赖关系
        -  通过键值观察监控NSOperation对象的属性
        -  指定某个操作的优先级
        -  重写NSOperation对象
    - `NSNotificationCenter`就是基于NSOperation实现的
- 44、dispatch_Group 分组
    - 将任务分组执行，可以有先后顺序的执行
    - dispatch_group_async(group,queue,block)
    - dispatch_group_enter(group)
    - dispatch_group_leave(group)
    - dispatch_group_wait(group,timeout)
    - dispatch_group_notify(group,queue,block)
    - dispatch_group_apply(count,queue,block) // 反复执行count次
- 45、dispatch_once 运行状态下只执行一次的代码
- 46、不要使用dispatch_get_current_queue
    - 很容易引起死锁
    - 只要是因为队列有层级关系
- 47、常见系统框架
    - *Foundation (OC)- NS****
    - *CoreFoundation (C)- 可以和Foundation进行桥接*
    - CFNetwork (OC)- BSD套接字封装成OC可用接口
    - CoreAudio (C)- 底层音频处理
    - AVFoundation (OC)- 音视频处理
    - CoreData (OC)- 数据持久化
    - CoreText (C)- 图文混排
    - CoreAnimation (OC) - 动画
    - CoreGraphics (C)- 2D渲染
    - MapKit - 地图
    - 不但要理解OC还要理解C。
- 48、多用block枚举，少用for循环
    - for循环
    - 使用NSEnumerator进行遍历
    - 使用for in 快速遍历
    - 基于块的遍历方式 
        - `enumeratObjectsUsingBlock:block`
        - `enumeratKeysAndObjectsUsingBlock:block`
        - `enumeratObjectsWithObjects:options UsingBlock:block` // options 指定遍历方式，枚举变量
    - 如果提前知道要遍历的集合类型，则在使用块遍历时可以修改方法签名，指出对象的具体类型。
- 49、对自定义内存管理的集合使用无缝桥接技术
    - Foundation和CoreFoundation 的无缝桥接
    - NSArray - CFArray

    ```objc
    NSArray *anNSArray = @[@1,@2,@3,@4]; // 对象
    CFArrayRef aCFArray = (__bridge CFArrayRef) anNSArray; // 数据结构
    NSLog(@"Count:%li",CFArrayGetCount(aCFArray));
    ```

    - OC 转换 C
        - __bridge表示这个对象的所有权归OC对象所有，所以会自动释放
        - __bridge_retained 则传递所有权交给其他对象，释放操作有接受的那个对象负责释放
    - C 转换 OC
        - __bridge_transfer OC对象获取所有权 
- 50、构建缓存时使用NSCache而非NSDictionary
    - NSPurgeableData 和 NSCache一起使用
- 51、精简initialize和load的代码
    -  load
        -  应用程序启动时会把用到的所有类的load方法都执行一遍
        -  加载顺序：首先是父类，然后是子类，最后是分类
    -  initialize
        -  运行时自动调用，懒加载方式调用
        -  线程安全调用
        -  子类没实现，调用父类的方法
- 52、别忘了NSTimer会保留其目标对象
    - 循环执行的定时器很容易出现循环引用，因为计时器会保留你目标对象，直到自己调用invalidate

