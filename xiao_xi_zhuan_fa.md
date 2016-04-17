# 消息转发



- 理解消息转发机制
	- 如果对象接收到未知的方法，会把这个方法转交给备用接收者，会调用方法`forwaringTargetorSelector:`如果这个方法还不能处理，就会进行完整的消息转发。
	- 消息转发机制分为两个步骤
	   - 第一、是在本类进行动态方法解析，会调用`+(BOOL)resolveInstanceMethod:(SEL)selector`或`+(BOOL)resolveClassMethod:(SEL)selector` ，其中`selector`就是那个位置的方法，可以在这里加入自己的处理逻辑，防止程序崩溃
	   - 第二、本类还有一次机会自己处理这个未知的方法，第一步没有效果的话，运行时会询问类是否进行消息转发`-(id)forwaringTargetorSelector:(SEL)selector`，`selector`表示未知的选择子，如果找到就返回新的接收对象，否则返回nil
	   - 第三，开启完整的消息转发，首先创建`NSInvocation`对象，把`selector`相关的信息包括接收者，方法名，和参数。这一步会调用`-(void)forwardInvocation:(NSInvocation) invocation `方法，然后逐层寻找，直到`NSObject`，如果还是没有，就调用`doesNotRecognizeSelector:`并抛出异常。
	- 可以在编写自己的类时，在消息转发过程中挂钩，用于执行预定的逻辑，而不是让程序崩溃掉。

## 自定义一个类

```objc
#import <Foundation/Foundation.h>

@interface SLQAutoDictionary : NSObject
/**<#注释#>*/
@property (nonatomic, strong) NSString *string;
/**<#注释#>*/
@property (nonatomic, strong) NSNumber *number;
/**<#注释#>*/
@property (nonatomic, strong) NSDate *date;
/**<#注释#>*/
@property (nonatomic, strong) id opaqueObject;
@end
```

## 实现文件

```objc
#import "SLQAutoDictionary.h"
#import <objc/runtime.h>

@interface SLQAutoDictionary ()
/**<#注释#>*/
@property (nonatomic, strong) NSMutableDictionary *backStore;
@end

@implementation SLQAutoDictionary

@dynamic string,number,date,opaqueObject;

- (instancetype)init
{
    self = [super init];
    if (self) {
        _backStore = [NSMutableDictionary new];
    }
    return self;
}

void autoDictionarySetter(id self,SEL _cmd,id value)
{
    NSLog(@"%@",value);
//    SLQAutoDictionary *typedSelf = (SLQAutoDictionary *)self;
//    NSMutableDictionary *backingStore = typedSelf.backStore;
//    NSString *key = NSStringFromSelector(_cmd);
}

id autoDictionaryGetter(id self,SEL _cmd)
{
     NSLog(@"getter");
    SLQAutoDictionary *typedSelf = (SLQAutoDictionary *)self;
    NSMutableDictionary *backingStore = typedSelf.backStore;
    NSString *key = NSStringFromSelector(_cmd);
    return [backingStore objectForKey:key];
}

void haha(id self,SEL _cmd,id value) {
    NSLog(@"haha");
}

id hahaha(id self,SEL _cmd){
    NSLog(@"hahaha");
    return @"getter hahaha";
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selectorString = NSStringFromSelector(sel);
    NSLog(@"selectorString：%@",selectorString);
    
    if ([selectorString hasPrefix:@"set"]) { // setter方法
       class_addMethod(self,
                       sel,
                       (IMP)haha,
                       "@:@");
    } else {
        class_addMethod(self,
                        sel,
                        (IMP)hahaha,
                        "@@:");
    }
    return YES;
}


@end
```


## 调用

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    SLQAutoDictionary *dict = [SLQAutoDictionary new];
    dict.string = @"test";
    NSLog(@"%@",dict.string);
    
}
```