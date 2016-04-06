# 获取当前view所在控制器


```objc
//
//  UIView+CurrentViewController.h
//  NanhaiPoliceM
//
//  Created by Christian on 16/4/6.
//  Copyright © 2016年 NanhaiPolice. All rights reserved.
//

#import <UIKit/UIKit.h>

@interface UIView (CurrentViewController)

/** 获取当前View的控制器对象 */
-(UIViewController *)getCurrentViewController;
@end


//
//  UIView+CurrentViewController.m
//  NanhaiPoliceM
//
//  Created by Christian on 16/4/6.
//  Copyright © 2016年 NanhaiPolice. All rights reserved.
//

#import "UIView+CurrentViewController.h"

@implementation UIView (CurrentViewController)

/** 获取当前View的控制器对象 */
- (UIViewController *)getCurrentViewController{
    UIResponder *next = [self nextResponder];
    do {
        if ([next isKindOfClass:[UIViewController class]]) {
            return (UIViewController *)next;
        }
        next = [next nextResponder];
    } while (next != nil);
    return nil;
}
@end
```