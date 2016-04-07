# 滚动视图-上下

# 顶部一个导航条，底部对应几个滚动视图

```objc
//
//  SLQHouseAddRenterViewController.h
//  NanhaiPoliceM
//
//  Created by Christian on 16/4/5.
//  Copyright © 2016年 Shenzhen Tentinet Technology Co,. Ltd. All rights reserved.
//

#import "BaseViewController.h"

/**
 *  人员界面类型
 */
typedef NS_ENUM(NSUInteger, SLQHouseAddRenterViewControllerType) {
    /**
     *  新增人员
     */
    SLQHouseAddRenterViewControllerTypeNew,
    /**
     *  人员详情
     */
    SLQHouseAddRenterViewControllerTypeDetail
};


@interface SLQHouseAddRenterViewController : BaseViewController
/**
 *  全能初始化方法
 *
 *  @param postType 类型
 *
 *  @return 返回值
 */
- (instancetype )initWithType:(SLQHouseAddRenterViewControllerType )type;
@end

```

# 实现文件

```objc

```

