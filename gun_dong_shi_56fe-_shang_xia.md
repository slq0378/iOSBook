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
//
//  SLQHouseAddRenterViewController.m
//  NanhaiPoliceM
//
//  Created by Christian on 16/4/5.
//  Copyright © 2016年 Shenzhen Tentinet Technology Co,. Ltd. All rights reserved.
//
#define SLQHouseAddRenterViewControllerInfoViewCount 3
#define SLQHouseInfoViewTag 1001

#import "SLQHouseAddRenterViewController.h"
#import "SLQSelectedButton.h"
#import "SLQLiveInfoView.h"
#import "SLQBasicInfoView.h"
#import "ZYQOtherInfoView.h"



@interface SLQHouseAddRenterViewController ()
<UIScrollViewDelegate>

/**控制器类型*/
@property (nonatomic, assign) SLQHouseAddRenterViewControllerType type;

/**选中按钮*/
@property (nonatomic, strong) UIButton *selectedBtn;
/**顶部scrollView*/
@property (nonatomic, strong) UIScrollView *horizontalTopScrollView;
/**topNavView*/
@property (nonatomic, strong) UIView *topNavView;
/**底部scrollView*/
@property (nonatomic, strong) UIScrollView *horizontalBottomScrollView;
/**居住信息*/
@property (nonatomic, strong) SLQLiveInfoView *liveInfoView;
/**基本信息*/
@property (nonatomic, strong) SLQBasicInfoView *basicInfoView;
/**其他信息*/
@property (nonatomic, strong) ZYQOtherInfoView *otherInfoView;
/**当前索引*/
@property (nonatomic, assign) NSInteger currentIndex;
@end

@implementation SLQHouseAddRenterViewController

#pragma mark - 控制器生命周期
/**
 *  重写默认初始化方法
 *
 *  @return self
 */
- (instancetype)init {
    if (self = [self initWithType:SLQHouseAddRenterViewControllerTypeNew]) {
        _type = SLQHouseAddRenterViewControllerTypeNew;
    }
    return self;
}

- (instancetype)initWithType:(SLQHouseAddRenterViewControllerType )type {
    if (self = [super init]) {
        _type = type;
    }
    return self;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    if (_type == SLQHouseAddRenterViewControllerTypeNew) {
        self.NaviTitle.text = @"新增人员";
    }else {
        self.NaviTitle.text = @"人员详情";
    }
    
    [self replaceRightButtonImage:nil Title:@"保存数据" clickSEL:@selector(save)];
    // 初始化界面
    [self setupUI];
}

/// 初始化界面
- (void)setupUI {
    self.horizontalTopScrollView.backgroundColor = [UIColor whiteColor];
    self.horizontalBottomScrollView.backgroundColor = [UIColor whiteColor];
    self.liveInfoView.backgroundColor = [UIColor whiteColor];
    self.basicInfoView.backgroundColor = [UIColor whiteColor];
    self.otherInfoView.backgroundColor = [UIColor whiteColor];
}

/**
 *  选中按钮，取消其他按钮选中状态
 *
 *  @param btn 选中按钮
 */
- (void)topNavViewSelectedBtn:(UIButton *)btn {
    
    for (UIView *view in _topNavView.subviews) {
        if ([view isKindOfClass:[SLQSelectedButton class]]) {
            [((SLQSelectedButton *)view) hideLine:YES];
        }
    }
    [((SLQSelectedButton *)btn) hideLine:NO];
    _selectedBtn = btn;
}

/**
 *  顶部导航条点击
 *
 *  @param btn 点击按钮，通过tag区分
 */
- (void)btnClick:(UIButton *)btn {
    switch (btn.tag) {
        case 1001: // 居住信息
        {
            [UIView animateWithDuration:0.3 animations:^{
                [self topNavViewSelectedBtn:btn];
                [self.horizontalBottomScrollView setContentOffset:CGPointMake(0, 0) animated:YES];
            }];
            
        }
            break;
        case 1002: // 基本信息
        {
            [UIView animateWithDuration:0.3 animations:^{
                [self topNavViewSelectedBtn:btn];
                [self.horizontalBottomScrollView setContentOffset:CGPointMake(1* ScreenWidth, 0) animated:YES];
            }];
        }
            break;
        case 1003: // 其他信息
        {
            [UIView animateWithDuration:0.3 animations:^{
                [self topNavViewSelectedBtn:btn];
                [self.horizontalBottomScrollView setContentOffset:CGPointMake(2* ScreenWidth, 0) animated:YES];
                
            }];
        }
            break;
        default:
            break;
    }
}

/**
 *  保存数据
 */
- (void)save {
    NSLog(@"%s",__FUNCTION__);
}

#pragma mark - UIScrollViewDelegate

-(void)scrollViewDidScroll:(UIScrollView *)scrollView
{
//    NSLog(@"%@",NSStringFromCGPoint(scrollView.contentOffset));
}

/**
 *@brief  UIScrollView一减速完成就会调用的方法
 *@param  UIScrollView类型
 *@return 无返回值
 */
-(void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    //    NSLog(@"开始拖拽");
    
    if (scrollView.tag == (SLQHouseInfoViewTag + 110))
    {
        
        NSInteger page = scrollView.contentOffset.x/[UIScreen mainScreen].bounds.size.width;
        _currentIndex = page;
        switch (page)
        {
            case 0: // 居住信息
            {
                
                [UIView animateWithDuration:0.15 animations:^{
                    // 改变顶部选中按钮
                    SLQSelectedButton *button = [_topNavView viewWithTag:SLQHouseInfoViewTag];
                    [self topNavViewSelectedBtn:button];
                    
                }];
                
            }
                break;
                
            case 1:// 基本信息
            {
                [UIView animateWithDuration:0.3 animations:^{
                    // 改变顶部选中按钮
                    SLQSelectedButton *button = [_topNavView viewWithTag:SLQHouseInfoViewTag + 1];
                    [self topNavViewSelectedBtn:button];
                }];
                
            }
                break;
                
            case 2:// 其他信息
            {
                [UIView animateWithDuration:0.15 animations:^{
                    // 改变顶部选中按钮
                    SLQSelectedButton *button = [_topNavView viewWithTag:SLQHouseInfoViewTag + 2];
                    [self topNavViewSelectedBtn:button];
                }];
                
            }
                break;
                
            default:
                break;
        }
        
    }
    
}

#pragma mark - 懒加载
/// 顶部scrollView
- (UIScrollView *)horizontalTopScrollView {
    if (!_horizontalTopScrollView) {
        // 顶部滚动条
        _horizontalTopScrollView = [[UIScrollView alloc] initWithFrame:CGRectMake(0*ScreenWidth, NavigationBarHeight, ScreenWidth,44)];
        //    horizontalTopScrollView.backgroundColor = [UIColor redColor];
        _horizontalTopScrollView.tag = SLQHouseInfoViewTag + 100; // 1001+100
        _horizontalTopScrollView.contentSize = CGSizeMake(ScreenWidth, 44);
        _horizontalTopScrollView.bounces = YES;
        _horizontalTopScrollView.delegate = self;
        _horizontalTopScrollView.pagingEnabled = YES;
        _horizontalTopScrollView.showsHorizontalScrollIndicator = NO;
        _horizontalTopScrollView.indicatorStyle = UIScrollViewIndicatorStyleWhite;
        [_horizontalTopScrollView addSubview:self.topNavView];
        [self.view addSubview:_horizontalTopScrollView];
    }
    return  _horizontalTopScrollView;
}
/// 底部scrollview
- (UIScrollView *)horizontalBottomScrollView {
    if (!_horizontalBottomScrollView) {
        //底部滚动条
        _horizontalBottomScrollView = [[UIScrollView alloc] initWithFrame:CGRectMake(0*ScreenWidth, CGRectGetMaxY(self.horizontalTopScrollView.frame), ScreenWidth, ScreenHeight - CGRectGetMaxY(self.horizontalTopScrollView.frame))];
        //    horizontalBottomScrollView.backgroundColor = [UIColor purpleColor];
        _horizontalBottomScrollView.tag = SLQHouseInfoViewTag + 110; // 1001+110
        _horizontalBottomScrollView.contentSize = CGSizeMake(3*ScreenWidth, ScreenHeight - CGRectGetMaxY(self.horizontalTopScrollView.frame));
        _horizontalBottomScrollView.bounces = YES;
        _horizontalBottomScrollView.delegate = self;
        _horizontalBottomScrollView.pagingEnabled = YES;
        _horizontalBottomScrollView.showsHorizontalScrollIndicator = NO;
        _horizontalBottomScrollView.indicatorStyle = UIScrollViewIndicatorStyleWhite;
        [self.view addSubview:_horizontalBottomScrollView];
    }
    return  _horizontalBottomScrollView;
}
/// 顶部导航栏（三个按钮）
- (UIView *)topNavView {
    if (!_topNavView) {
        // 导航条
        _topNavView = [[UIView alloc] initWithFrame:CGRectMake(0, 0,ScreenWidth, 44)];
        NSArray *title = @[@"居住信息",@"基本信息",@"其他信息"];
        for(NSInteger i = 0 ; i < title.count; i ++) {
            SLQSelectedButton *btn = [[SLQSelectedButton alloc] initWithFrame:CGRectMake(i * ScreenWidth/SLQHouseAddRenterViewControllerInfoViewCount, 0, ScreenWidth/SLQHouseAddRenterViewControllerInfoViewCount, 44)];
            btn.tag = SLQHouseInfoViewTag + i;
            [btn setTitle:title[i] forState:UIControlStateNormal];
            [btn addTarget:self action:@selector(btnClick:) forControlEvents:UIControlEventTouchUpInside];
            
            [_topNavView addSubview:btn];
            if (i == 0 ) {
                [self topNavViewSelectedBtn:btn];
            }
        }
    }
    return  _topNavView;
}
/// 居住信息
- (SLQLiveInfoView *)liveInfoView {
    if (!_liveInfoView) {
        _liveInfoView = [[SLQLiveInfoView alloc] initWithFrame:CGRectMake(0*ScreenWidth, 0, ScreenWidth, ScreenHeight - CGRectGetMaxY(self.horizontalTopScrollView.frame))];
        [self.horizontalBottomScrollView addSubview:_liveInfoView];
    }
    return  _liveInfoView;
}
/// 基本信息
- (SLQBasicInfoView *)basicInfoView {
    if (!_basicInfoView) {
        _basicInfoView = [[SLQBasicInfoView alloc] initWithFrame:CGRectMake(1*ScreenWidth, 0, ScreenWidth, ScreenHeight - CGRectGetMaxY(self.horizontalTopScrollView.frame))];
        [self.horizontalBottomScrollView addSubview:_basicInfoView];
        
    }
    return  _basicInfoView;
}
/// 其他信息
- (ZYQOtherInfoView *)otherInfoView {
    if (!_otherInfoView) {
        _otherInfoView = [[ZYQOtherInfoView alloc] initWithFrame:CGRectMake(2*ScreenWidth, 0, ScreenWidth, ScreenHeight - CGRectGetMaxY(self.horizontalTopScrollView.frame))];
        [self.horizontalBottomScrollView addSubview:_otherInfoView];
    }
    return  _otherInfoView;
}
@end
```

