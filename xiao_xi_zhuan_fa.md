# 消息转发



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