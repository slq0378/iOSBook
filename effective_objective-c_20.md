# Effective OC 2.0

- 1、OC的特殊之处
	- 函数调用语言的代码执行由编译器决定，如果是多态的话，则通过虚函数表在运行时再决定调用那个函数。
	- 消息结构语言函数的调用只会在运行时才会寻找要调用的函数。
- 2、将引入头文件的时机尽量后延，只有在明确需要时才引入。
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
	- 使用const 常量可以利用一些编译器特性
	- 如果准备在外部也使用这个常量，需要这样定义

	```objc
	//.h
	extern const NSTimeInterval ClassName+AnimationDuraton;
	//.m
	const NSTimeInterval ClassName+AnimationDuraton = 3.0;
	```
	
- 5、用枚举表示状态、选项、状态码 
- 6、属性 理解
- 7、在对象内部尽量直接访问实例变量
	- 点语法访问：经过getter方法
	- 直接访问：不经过getter方法，速度更快，不会触发KVO
	- **解决方法**：读取数据，直接访问，写入数据，属性（点语法）访问，在初始化方法和dealloc中，始终使用直接访问方式。
- 8、理解“对象等同性”这一概念
	- 若两个对象相等，那么hash码也相等
	- `==` 指针比较
	- `isEqual`
	- `isEqualToString`
	- `hash`
- 9、以“类族模式”隐藏实现细节
- 10、在既有类中使用关联对象存放自定义数据
	- 1、可以通过“关联机制”将两个对象关联起来
	- 2、关联时要指名拥有关系
	- 3、尽量避免使用关联，因为出现bug，很难调试
- 11、理解`objc_msgSend`的作用
	- 消息发送机制 
	- 查找过程：接收者类缓存->类方法列表->父类方法->找不到就进行消息转发
- 12、理解消息转发机制
	- 如果对象接收到未知的方法，会把这个方法转交给备用接收者，会调用方法`forwaringTargetorSelector:`如果这个方法还不能处理，就会进行完整的消息转发。
	- 首先创建NSInvocation对象，把selector相关的信息包括接收者，方法名，和参数。这一步会调用forwardInvocation: 方法，然后逐层寻找，直到NSObject，如果还是没有，就调用`doesNotRecognizeSelector:`并抛出异常。
