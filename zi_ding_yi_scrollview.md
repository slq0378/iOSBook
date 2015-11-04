# 自定义scrollView

```objc
//
//  ChatbarBroadcastView.h
//  iAround
//
//  Created by songlq on 15/10/22.
//   聊吧广播
//

#import <UIKit/UIKit.h>


@class ChatbarInfo;
@class ZGImageView;

/**
 *  聊吧广播类型
 */
typedef NS_ENUM(NSUInteger, ChatbarBroadcastViewType) {
    /**
     *  聊吧大厅广播
     */
    ChatbarBroadcastViewTypeChatbar,
    /**
     *  聊吧面板广播
     */
    ChatbarBroadcastViewTypeChatpanel,
};

/**
 *  点击按钮，跳转控制器类型
 */
typedef NS_ENUM(NSUInteger, ChatbarScrollViewButtonType) {
    /**
     *  用户信息面板
     */
    ChatbarScrollViewButtonTypeUser,
    /**
     *  聊吧面板
     */
    ChatbarScrollViewButtonTypeChatbar,
};


@protocol ChatbarBroadcastViewDelegate <NSObject>

@optional
/**
 *  传递用户按钮点击
 *
 *  @param userid 用户id
 */
- (void)ChatbarBroadcastViewDidClickUserButtonWithUserid:(long long)userid;
/**
 *  传递聊吧按钮点击
 *
 *  @param chatInfo 聊吧信息模型
 */
- (void)ChatbarBroadcastViewDidClickChatbatButtonWithChatInfo:(ChatbarInfo *)chatInfo;

@end


@interface ChatbarBroadcastView : UIView

@property (nonatomic,assign) id<ChatbarBroadcastViewDelegate> delegate;

/**
 *  自动滚动广播视图
 *
 *  @param broadcastType 广播类型
 */
- (void)setAutoScrollWithType:(ChatbarBroadcastViewType )broadcastType;

/** 更新聊吧广播信息 */
- (void)updateBroadcastInfo;
@end


#pragma mark - 封装广播子视图
/**
 *  封装广播视图
 */
@protocol ChatbarScrollViewDelegate <NSObject>

@optional
/**
 *  传递用户按钮点击事件
 *
 *  @param userid 用户id
 */
- (void)ChatbarScrollViewDidClickUserButtonWithUserid:(long long)userid;
/**
 *  传递聊吧名称点击事件
 *
 *  @param chatInfo 聊吧信息模型
 */
- (void)ChatbarScrollViewDidClickChatbatButtonWithChatInfo:(ChatbarInfo *)chatInfo;
@end

@interface ChatbarScrollView: UIView

@property (nonatomic,assign) id<ChatbarScrollViewDelegate> delegate;
/**
 *  设置聊吧面板
 *
 *  @param chatbarInfo 聊吧面板模型
 */
- (void)setupCharpanelViewWithInfo:(ChatbarInfo *)chatbarInfo;
/**
 *  设置聊吧大厅
 *
 *  @param chatbarInfo 聊吧大厅模型
 */
- (void)setupChatbarViewWithInfo:(ChatbarInfo *)chatbarInfo;
@end
```

```objc
//
//  ChatbarBroadcastView.m
//  iAround
//
//  Created by songlq on 15/10/22.
//  聊吧广播
//

#import "ChatbarBroadcastView.h"
#import "ZGImageView.h"
#import "U6AppFacade.h"
#import "SummaryInfoProxy.h"
#import "U6AppFacade+Proxy.h"
#import "chatbarInfo.h"
#import "ChatbarHallViewController.h"
#import "ChatPanelViewController.h"

#if !__has_feature(objc_arc)
#error no "objc_arc" compiler flag
#endif

#define ScreenWidth [UIScreen mainScreen].bounds.size.width

@interface ChatbarBroadcastView() <ChatbarScrollViewDelegate>

@end

@implementation ChatbarBroadcastView
{
    /** 广播背景 */
    UIView *_bgIconView;
    /** 广播图标 */
    ZGImageView *_iconView;
    /** 滚动条 */
    UIView *_scrollView;
    
    /** 广播数组，可能为空，通过通知来更新这个数组 */
    NSMutableArray *_broadcastArray;
    
    /** 滚动定时器 */
    CADisplayLink *_scrollTimer;
    
    ChatbarBroadcastViewType _type; // 广播类型
    
    NSInteger _currentScrollViewIndex; // 当前滚动的广播
    
}

- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        
        _currentScrollViewIndex = 0;
        // 滚动条
        UIView *scorllView = [[UIView alloc] initWithFrame:CGRectMake(64, 0, frame.size.width, frame.size.height)];
        _scrollView = scorllView;
        [self addSubview:_scrollView];
        _scrollView.userInteractionEnabled = YES;
        
        _broadcastArray = [[NSMutableArray alloc] init];
        // 初始化子控件
        ZGImageView *icon = [[ZGImageView alloc] init];
        _iconView = icon;
        _bgIconView = [[UIView alloc] init];
        [self insertSubview:_bgIconView aboveSubview:_scrollView];
        [self insertSubview:_iconView aboveSubview:_bgIconView]; // 添加到最顶层
        // 添加定时器 滚动显示
        _scrollTimer = [CADisplayLink displayLinkWithTarget:self selector:@selector(scrollView)];
        //        _scrollTimer.frameInterval = 2; // 刷新频率30Hz
        [_scrollTimer addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
        
    }
    return self;
}
/** 更新聊吧广播信息 */
- (void)updateBroadcastInfo
{
    // 更新广播
    // 清空广播数组
    [_broadcastArray removeAllObjects];
    // 移除当前正在显示的广播
    [_scrollView removeAllSubviews];
    _scrollView.frame = CGRectZero;
    //
    [self setupScrollView];
}
/** 自动滚动广播视图 */
- (void)setAutoScrollWithType:(ChatbarBroadcastViewType)broadcastType
{
    _type = broadcastType;
    
//    [NSTimer scheduledTimerWithTimeInterval:1.0f
//                                     target:self
//                                   selector:@selector(setupScrollView)
//                                   userInfo:nil repeats:NO];
    [self updateBroadcastInfo];
}
/** 初始化滚动条信息 */
- (void)setupScrollView
{
    // 获取用户聊吧信息
    SummaryInfoProxy *summaryInfo = [[U6AppFacade getInstance] getSummaryInfo];
    NSArray *broadcastInfo = summaryInfo.chatbarAdArr;
    
    // 如果广播数组为空，就不显示控件
    if (0 == broadcastInfo.count)
    {
        self.hidden = YES;
        return;
    }
    
    self.hidden = NO;
    // 多条广播
    NSInteger broadcastCount = broadcastInfo.count;
    for (NSInteger i = 0 ; i < broadcastCount; i ++)
    {
        ChatbarInfo *chatbarInfo = broadcastInfo[i];
        // 聊吧面板
        if (_type == ChatbarBroadcastViewTypeChatpanel) {
            [self setupCharpanelViewWithInfo:chatbarInfo];
        }
        // 聊吧大厅
        else if (_type == ChatbarBroadcastViewTypeChatbar){
            [self setupChatbarViewWithInfo:chatbarInfo];
        }
    }
    
}
/** 初始化聊吧面板广播 */
- (void)setupCharpanelViewWithInfo:(ChatbarInfo *)chatbarInfo
{
    self.backgroundColor = RGBACOLOR(0, 0, 0, 0.7f);
    CGRect frame = self.frame;
    //
    _iconView.frame = CGRectMake(11, 6, 13, 13);
    _iconView.image = [UIImage imageNamed:@"chatpanel_laba"];
    
    _bgIconView.backgroundColor = RGBACOLOR(0, 0, 0, 1.0f);
    _bgIconView.frame = CGRectMake(0, 0, 29, 24);


    // 滚动条
    _scrollView.frame = CGRectMake(_iconView.right + 5, 0, self.width , self.height );
    _scrollView.center = CGPointMake(_scrollView.center.x, _iconView.center.y);
    
    // 创建广播
    ChatbarScrollView *scroll = [[ChatbarScrollView alloc] initWithFrame:_scrollView.frame]; //
    scroll.delegate = self;
    [_scrollView addSubview:scroll];
    
    [scroll setupCharpanelViewWithInfo:chatbarInfo];
    if (_scrollView.subviews.count == 1)
    {
        scroll.frame = CGRectMake(0, _scrollView.top, scroll.width, scroll.height); // 确定位置和尺寸
    }
    else
    {
        scroll.frame = CGRectMake(_scrollView.right, _scrollView.top, scroll.width, scroll.height); // 确定位置和尺寸
    }
    scroll.center = CGPointMake(scroll.center.x, _iconView.center.y);
    [_broadcastArray addObject:scroll];
    // 重新设置_scrollView的frame
    _scrollView.frame = CGRectMake(_iconView.right + 4, 0, scroll.right, frame.size.height); // 改变宽度
    // 重新设置当前控件的frame
    self.frame = CGRectMake(self.left, self.top, _scrollView.right, _scrollView.height);
}

/** 初始化聊吧大厅广播 */
- (void)setupChatbarViewWithInfo:(ChatbarInfo *)chatbarInfo
{
    self.backgroundColor = [UIColor whiteColor];
    CGRect frame = self.frame;
    //
    _iconView.image = [UIImage stretchableImageName:@"chatbar_guangbo"];
    _iconView.frame = CGRectMake(15,(self.height - 28)/2 , 28, 28);

    _bgIconView.backgroundColor = [UIColor whiteColor];
    _bgIconView.frame = CGRectMake(0, 0, 48, 38);
    
    
    _scrollView.frame = CGRectMake(_iconView.right + 8, 0, self.width, self.height); // 初始化位置和尺寸
    _scrollView.center = CGPointMake(_scrollView.center.x, _iconView.center.y);
    
    // 创建广播
    ChatbarScrollView *scroll = [[ChatbarScrollView alloc] initWithFrame:_scrollView.frame]; //
    scroll.delegate = self;
    [_scrollView addSubview:scroll];
    
    [scroll setupChatbarViewWithInfo:chatbarInfo];
    if (_scrollView.subviews.count == 1)
    {
        scroll.frame = CGRectMake(0, _scrollView.top, scroll.width, scroll.height); // 确定位置和尺寸
    }
    else
    {
        scroll.frame = CGRectMake(_scrollView.right, _scrollView.top, scroll.width, scroll.height); // 确定位置和尺寸
    }
    scroll.center = CGPointMake(scroll.center.x, _iconView.center.y);
    [_broadcastArray addObject:scroll];
    
    _scrollView.frame = CGRectMake(_iconView.right + 8, 0, scroll.right, frame.size.height); // 改变宽度
    
    self.frame = CGRectMake(self.left, self.top, _scrollView.right, _scrollView.height);
    
}

/** 滚动视图 */
- (void)scrollView
{
    NSInteger count = _broadcastArray.count;
    if (_broadcastArray.count <=0 || _scrollView.subviews.count <= 0)
        return;
    
    ChatbarScrollView *scroll = (ChatbarScrollView *)_scrollView.subviews[_currentScrollViewIndex];
    
    __block CGRect rect = scroll.frame;
//    scroll.frame = CGRectMake(YJ_WIDTH, rect.origin.y, rect.size.width, rect.size.height);
     NSInteger nextScroll = _currentScrollViewIndex;
//    [UIView setAnimationsEnabled:YES];
    [UIView animateWithDuration:0.3f delay:0 options:UIViewAnimationOptionAllowUserInteraction animations:^{
        rect.origin.x -= 1;
        
        // 移动到最后，就对scroll的frame重新计算
        if (rect.origin.x == - (scroll.width)/2) {
//            [UIView setAnimationsEnabled:NO];
            _currentScrollViewIndex = (_currentScrollViewIndex + 1) % count;
//            ChatbarScrollView *nextScroll = (ChatbarScrollView *)_scrollView.subviews[_currentScrollViewIndex];

            scroll.frame = CGRectMake(YJ_WIDTH  + 10, rect.origin.y, rect.size.width, rect.size.height);
        }
        scroll.frame = rect;
    } completion:^(BOOL finished) {
        
    }];

    
//    __block CGRect rect = _scrollView.frame;
//    [UIView setAnimationsEnabled:YES];
//    [UIView animateWithDuration:0.3f delay:0 options:UIViewAnimationOptionAllowUserInteraction animations:^{
//        rect.origin.x -= 1;
//        
//        // 移动到最后，就对_scrollView的frame重新计算
//        if (rect.origin.x <= - (_scrollView.width)) {
//            
//            [UIView setAnimationsEnabled:NO];
//            [UIView animateWithDuration:0.2f delay:0 options:UIViewAnimationOptionTransitionFlipFromRight | UIViewAnimationOptionAllowUserInteraction animations:^{
//                rect.origin.x = 60;
//            } completion:^(BOOL finished) {
//                rect.origin.x =  50;
//            }];
//        }
//        _scrollView.frame = rect;
//    } completion:^(BOOL finished) {
//        
//    }];
}

#pragma mark  实现代理方法
/** 点击用户按钮 */
- (void)ChatbarScrollViewDidClickUserButtonWithUserid:(long long)userid
{
    if ([self.delegate respondsToSelector:@selector(ChatbarBroadcastViewDidClickUserButtonWithUserid:)])
    {
        [self.delegate ChatbarBroadcastViewDidClickUserButtonWithUserid:userid];
    }
}
/** 点击聊吧按钮 */
- (void)ChatbarScrollViewDidClickChatbatButtonWithChatInfo:(ChatbarInfo *)chatInfo
{
    if ([self.delegate respondsToSelector:@selector(ChatbarBroadcastViewDidClickChatbatButtonWithChatInfo:)])
    {
        [self.delegate ChatbarBroadcastViewDidClickChatbatButtonWithChatInfo:chatInfo];
    }
}

- (void)dealloc
{
    [_scrollTimer invalidate];
    _scrollTimer = nil;
}

@end

#pragma mark - 广播子视图实现
@implementation ChatbarScrollView
{
    /** 用户名 */
    UIButton *_userName;
    /**  招募新人或管理员*/
    UILabel *_recruitLabel;
    /** 聊吧名称 */
    UIButton *_chatbarName;
    /**  聊吧标签 */
    UILabel *_chatbarLabel;
    
    /** 用户id */
    long long _userid;
    /** 聊吧信息 */
    ChatbarInfo *_chatbarInfo;
    /** 招募文本 */
    NSArray *_recruitArr;
}


- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        
        // 按钮点击
        UIButton *userButton = [[UIButton alloc] init];
        [userButton addTarget:self action:@selector(userButtonClick:) forControlEvents:UIControlEventTouchUpInside];
        _userName = userButton;
        [self addSubview:_userName];
        
        // 邀请你加入
        UILabel *inviteYouLabel = [[UILabel alloc] init];
        _recruitLabel = inviteYouLabel;
        [self addSubview:inviteYouLabel];
        
        // 聊吧名称
        UIButton *chatbarButton = [[UIButton alloc] init];
        [chatbarButton addTarget:self action:@selector(chatbarButtonClick:) forControlEvents:UIControlEventTouchUpInside];
        _chatbarName = chatbarButton;
        
        [self addSubview:_chatbarName];
        
        // 聊吧文字
        UILabel *chatbarLabel = [[UILabel alloc] init];
        _chatbarLabel = chatbarLabel;
        _chatbarLabel.backgroundColor = [UIColor clearColor];
        [self addSubview:chatbarLabel];
        
        // 滚动条
        self.frame = CGRectMake(frame.origin.x, 0, _chatbarLabel.right , frame.size.height);
        
        _recruitArr = [[NSArray alloc] initWithObjects:
                       NSLocalizedString(@"CHATPANEL_BROADCAST_RECRUIT_NEWGUY1", @"快来加入我们的聊吧"),
                       NSLocalizedString(@"CHATPANEL_BROADCAST_RECRUIT_NEWGUY2", @"缺的就是你，加入我们"),
                       NSLocalizedString(@"CHATPANEL_BROADCAST_RECRUIT_NEWGUY3", @"小伙伴们都在这里了!"),
                       NSLocalizedString(@"CHATPANEL_BROADCAST_RECRUIT_ADMINISTRATOR", @"招募管理员!"),
                       NSLocalizedString(@"CHATPANEL_BROADCAST_BAR", @"吧!"),
                       nil];
        
    }
    return self;
}
// 设置广播类型
- (void)setupBroadcastType:(ChatbarInfo *)chatbarInfo
{
    /*
     5.招募新人广播文本，随机使用其中之一：
     1）吼吼女王（用户名） 快来加入我们的聊吧 广州相亲队（吧名称） 吧！；
     2）吼吼女王（用户名） 缺的就是你，加入我们 广州相亲队（吧名称） 吧！；
     3）吼吼女王（用户名） 广州相亲队（吧名称） 小伙伴们都在这里了！
     6.招募管理员文本：
     吼吼女王（用户名） 广州相亲队（吧名称） 招募管理员！
     */
    
    // 1:招募新人,2:招募管理员
    if (chatbarInfo.adtype == 1) { // 招募新人
        NSInteger index = arc4random_uniform(3); // 如果等于2就设置_chatbarLabel，否则就设置_recruitLabel
        if (2 == index) {
            _recruitLabel.text = @"";
            _chatbarLabel.text = _recruitArr[2];
        }else
        {
            _recruitLabel.text = _recruitArr[index];
            _chatbarLabel.text = _recruitArr[4];;
        }
    }
    else if(chatbarInfo.adtype == 2) { // 招募管理员
        _recruitLabel.text = @"";
        _chatbarLabel.text = _recruitArr[3];
    }
    
}

/** 初始化聊吧面板广播 */
- (void)setupCharpanelViewWithInfo:(ChatbarInfo *)chatbarInfo
{
    self.backgroundColor = [UIColor clearColor];
    // 保存用户id和聊吧信息
    _userid = chatbarInfo.userid;
    _chatbarInfo = chatbarInfo;
    
    // 设置广播内容
    [self setupBroadcastType:chatbarInfo];
    
    // 按钮点击
    [_userName setTitleColor:UIColorFromRGB(0xFFFFFF)];
    [_userName.titleLabel setFont:[UIFont systemFontOfSize:11.0f]];
    [_userName setTitle:[NSString stringWithFormat:@"%@：",chatbarInfo.username] forState:UIControlStateNormal];
    [_userName sizeToFit];
    _userName.frame = CGRectMake(0, 0, _userName.width, self.height);
    _userName.backgroundColor = [UIColor clearColor];
    
    // 推荐加入
    
    _recruitLabel.font = [UIFont systemFontOfSize:11.0f];
    _recruitLabel.textColor = UIColorFromRGB(0xFFFFFF);
    [_recruitLabel sizeToFit];
    _recruitLabel.frame = CGRectMake(_userName.right + 4, 0, _recruitLabel.width, self.height);
    _recruitLabel.center = CGPointMake(_recruitLabel.center.x, _userName.center.y);
    _recruitLabel.backgroundColor = [UIColor clearColor];
    
    // 聊吧名称
    [_chatbarName setTitleColor:UIColorFromRGB(0xCC4E56)];
    [_chatbarName.titleLabel setFont:[UIFont systemFontOfSize:11.0f]];
    [_chatbarName setTitle:_chatbarInfo.nameStr forState:UIControlStateNormal];
    [_chatbarName sizeToFit];
    UIView *bottomLine = [[UIView alloc] init];
    bottomLine.backgroundColor = UIColorFromRGB(0xef555e);
    [_chatbarName addSubview:bottomLine];
    _chatbarName.frame = CGRectMake(_recruitLabel.right + 4, 0, _chatbarName.width, self.height);
    _chatbarName.backgroundColor = [UIColor clearColor];
    bottomLine.frame = CGRectMake(0, _chatbarName.titleLabel.frame.origin.y +  _chatbarName.titleLabel.height - 1, _chatbarName.width, 1);
    // 聊吧文字
    _chatbarLabel.font = [UIFont systemFontOfSize:11.0f];
    _chatbarLabel.textColor = UIColorFromRGB(0xFFFFFF);
    [_chatbarLabel sizeToFit];
    _chatbarLabel.frame = CGRectMake(_chatbarName.right + 4, 0, _chatbarLabel.width, self.height);
    _chatbarLabel.center = CGPointMake(_chatbarLabel.center.x, _chatbarName.center.y);
    _chatbarLabel.backgroundColor = [UIColor clearColor];
    
    self.frame = CGRectMake(self.left, 0, _chatbarLabel.right, self.height);
}
/** 初始化聊吧大厅广播 */
- (void)setupChatbarViewWithInfo:(ChatbarInfo *)chatbarInfo
{
    // 保存用户id和聊吧信息
    _userid = chatbarInfo.userid;
    _chatbarInfo = chatbarInfo;
    
    // 更新内容
    [self setupBroadcastType:chatbarInfo];
    
    // 按钮点击
    // 男 2a9bfc 女 ff60b9
    if ([chatbarInfo.genderString isEqualToString:@"m"]) {
        [_userName setTitleColor:UIColorFromRGB(0x2a9bfc)];
    }else if ([chatbarInfo.genderString isEqualToString:@"f"]) {
        [_userName setTitleColor:UIColorFromRGB(0xff60b9)];
    }else { // 默认显示为男性
        [_userName setTitleColor:UIColorFromRGB(0x2a9bfc)];
    }
    [_userName.titleLabel setFont:[UIFont systemFontOfSize:13.0f]];
    [_userName setTitle:[NSString stringWithFormat:@"%@：",chatbarInfo.username] forState:UIControlStateNormal];
    [_userName sizeToFit];
    _userName.frame = CGRectMake(0, 0, _userName.width, self.height);
    
    // 邀请你加入
    _recruitLabel.font = [UIFont systemFontOfSize:13.0f];
    _recruitLabel.textColor = COLOR_333333;
    [_recruitLabel sizeToFit];
    _recruitLabel.frame = CGRectMake(_userName.right + 4, 0, _recruitLabel.width, self.height);
    _recruitLabel.center = CGPointMake(_recruitLabel.center.x, _userName.center.y);
    
    // 聊吧名称
    [_chatbarName setTitleColor:UIColorFromRGB(0xef555e)];
    [_chatbarName.titleLabel setFont:[UIFont systemFontOfSize:13.0f]];
    [_chatbarName setTitle:_chatbarInfo.nameStr forState:UIControlStateNormal];
    UIView *bottomLine = [[UIView alloc] init];
    bottomLine.backgroundColor = UIColorFromRGB(0xef555e);
    [_chatbarName addSubview:bottomLine];
    [_chatbarName.titleLabel sizeToFit];
    [_chatbarName sizeToFit];
    _chatbarName.frame = CGRectMake(_recruitLabel.right + 4, 0, _chatbarName.width, self.height);
    bottomLine.frame = CGRectMake(0, _chatbarName.titleLabel.frame.origin.y +  _chatbarName.titleLabel.height - 1, _chatbarName.width, 1);
    
    // 聊吧文字
    _chatbarLabel.font = [UIFont systemFontOfSize:13.0f];
    _chatbarLabel.textColor = COLOR_333333;
    [_chatbarLabel sizeToFit];
    _chatbarLabel.frame = CGRectMake(_chatbarName.right + 4, 0, _chatbarLabel.width, self.height);
    _chatbarLabel.center = CGPointMake(_chatbarLabel.center.x, _chatbarName.center.y);
    
    self.frame = CGRectMake(self.left, 0, _chatbarLabel.right, self.height);
}

/** 用户按钮点击 */
- (void)userButtonClick:(UIButton *)userBtn
{
    YJLog(@"%s",__func__);
    // 进入用户信息界面,传递消息
    if ([self.delegate respondsToSelector:@selector(ChatbarScrollViewDidClickUserButtonWithUserid:)])
    {
        [self.delegate ChatbarScrollViewDidClickUserButtonWithUserid:_userid];
    }
}
/** 聊吧按钮点击 */
- (void)chatbarButtonClick:(UIButton *)chatbarBtn
{
    YJLog(@"%s",__func__);
    // 进入聊吧
    if ([self.delegate respondsToSelector:@selector(ChatbarScrollViewDidClickChatbatButtonWithChatInfo:)])
    {
        [self.delegate ChatbarScrollViewDidClickChatbatButtonWithChatInfo:_chatbarInfo];
    }
}

@end
```
- 测试
- 这次的滚动没有问题，通过两个定时器来控制器两个View的滚动，
- `clipsToBounds` 这个属性控制超过边界不显示
```objc
//
//  ViewController.m
//  SLQScrollView
//
//  Created by songlq on 15/10/27.
//  Copyright (c) 2015年 songlq. All rights reserved.
//  滚动条

#import "ViewController.h"


@interface ViewController ()

@property (nonatomic,strong) CADisplayLink *displayLink;
@property (nonatomic,strong) CADisplayLink *displayLink2;

/**滚动条*/
@property (nonatomic, weak) UIView *scrollView;
/**索引*/
@property (nonatomic, assign) NSInteger index;
/**索引*/
@property (nonatomic, assign) NSInteger next;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self setupScrollView];
}
/**
 *  初始化scrollView
 */
- (void)setupScrollView
{
    UIView *scroll = [[UIView alloc] initWithFrame:CGRectMake(20, 100, 200, 100)];
    self.scrollView = scroll;
    [self.view addSubview:self.scrollView];
    self.scrollView.backgroundColor = [UIColor orangeColor];
    
    // 设置基本属性
    NSArray *colors = @[[UIColor redColor],[UIColor blueColor],[UIColor greenColor]];
    
    for(NSInteger i = 0 ; i < 3 ; i++)
    {
        UIView *view = [[UIView alloc] init];
        UILabel *label = [[UILabel alloc] init];
        [view addSubview:label];
        
        if ( 0 == i) { // 第一个要特别设置
            view.frame = CGRectMake(0, 0, 200, 44);
        }
        else
        {
            view.frame = CGRectMake(200, 0, 250, 44);
        }
        label.frame = CGRectMake(0, 0, 100, 44);
        view.backgroundColor = colors[i];
        
        label.text =  [NSString stringWithFormat:@"%zd啦啦",i+1];
        label.backgroundColor = [UIColor grayColor];
        UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tapClick)];
        [label addGestureRecognizer:tap];
        label.userInteractionEnabled = YES;
        [self.scrollView addSubview:view];
        self.scrollView.clipsToBounds = YES; // 超出控件frame不显示
    }
    
    self.displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(update)];
    [self.displayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
    
    self.displayLink2 = [CADisplayLink displayLinkWithTarget:self selector:@selector(update2)];
    [self.displayLink2 addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
    self.displayLink2.paused = YES;
}

-(void)tapClick
{
    NSLog(@"****************************");
}
- (void)update2
{
    NSInteger count = self.scrollView.subviews.count;
    if (count == 0) {
        return;
    }
    UIView *view = self.scrollView.subviews[self.next]; // 取出View
    CGRect rect = view.frame;
    rect.origin.x -- ;
    view.frame = rect;
    NSLog(@"next22222----%zd",self.next);
    if (view.frame.origin.x == -(view.frame.size.width)/2) {
        rect.origin.x -- ;
        view.frame = rect;
        self.index = (self.next + 1)%count;
        self.displayLink.paused = NO;
    }
    else if (view.frame.origin.x == -(view.frame.size.width)) {
        // 恢复自己的位置
        rect.origin.x  = view.frame.size.width;
        view.frame = rect;
        self.displayLink2.paused = YES;
    }
}
- (void)update
{
    NSInteger count = self.scrollView.subviews.count;
    if (count == 0) {
        return;
    }
    UIView *view = self.scrollView.subviews[self.index]; // 取出View

    CGRect rect = view.frame;
    rect.origin.x -- ;
    view.frame = rect;
    NSLog(@"index11111----%zd",self.index);
    if (view.frame.origin.x == -(view.frame.size.width)/2) {
        rect.origin.x -- ;
        view.frame = rect;
        self.next = (self.index + 1)%count;
        self.displayLink2.paused = NO;
    }
    else if (view.frame.origin.x == -(view.frame.size.width)) {
        // 开启第二个定时器
        // 恢复自己的位置
        rect.origin.x  = view.frame.size.width ;
        view.frame = rect;
        self.displayLink.paused = YES;
        
    }
}

- (void)exchangeView:(UIView *)first withView:(UIView *)second
{
    CGRect firstFrame = first.frame;
    CGRect secondFrame = second.frame;
    first.frame = secondFrame;
    second.frame = firstFrame;
    [self.scrollView setNeedsDisplay];
}

- (void)dealloc
{
    [self.displayLink invalidate];
    [self.displayLink2 invalidate];
    self.displayLink2 = nil;
    self.displayLink = nil;
}

@end

```