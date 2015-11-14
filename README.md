# iOSBook
//
//  ChatPanelViewController.m
//  iAround
//
//  Created by gaoyc on 15/9/21.

#import "ChatPanelViewController.h"
#import "ChatPanelVoiceMessageView.h"
#import "ChatPanelPublicMessageView.h"
#import "ChatbarHallViewController.h"
#import "ChatbarProxy.h"
#import "U6AppFacade+Proxy.h"
#import "MessageInfo.h"
#import "YJChatPanelInputView.h"
#import "U6Common.h"
#import "KeywordProxy.h"
#import "YJTitleView.h"
#import "ChatbarInfoViewController.h"

#import "ChatPanelGiftSheetView.h"
#import "ChatPanelActionSheet.h"
#import "ChatPanelSkillSheetView.h"
#import "ChatbarBroadcastView.h"

#import "ChatPanelViewController.h"
#import "UserInfoViewController.h"
#import "U6AppFacade+ChatbarBiz.h"
#import "ChatbarInfoMemberViewController.h"

#define PARAM_KEY_SOUND_FILENAME @"soundFileName"
#define PARAM_KEY_SOUND_TIME @"sound_time"
#define PARAM_KEY_SEND_FLAG @"send_flag"

#define PUBLIC_MESSAGE_PZGESIZE 50
#define VOICE_MESSAGE_PAGESIZE 10

#if !__has_feature(objc_arc)
#error no "objc_arc" compiler flag
#endif

@interface ChatPanelViewController ()
<YJChatPanelInputViewDelegate,
ChatPanelPublicMessageViewDelegate,
ChatPanelVoiceMessageViewDelegate,
ChatbarBroadcastViewDelegate,
ChatPanelGiftSheetViewDelegate,
YJTitleViewDelegate
>
{
    ChatbarInfo *_chatbarInfo;
    
    UIButton *_rightButton;
    YJTitleView *_titleView;
    
    ChatPanelVoiceMessageView *_voiceView;
    ChatPanelPublicMessageView *_publicView;
    
    ChatbarProxy *_chatbarProxy;
    YJChatPanelInputView *_inputView;
    KeywordProxy *_keywordProxy;
    
    ChatbarBroadcastView *_chatbarBroadcastView; //聊吧面板广播视图
    
    NSMutableArray *_publicMessageArr;
    NSMutableArray *_voiceMessageArr;
    long long _lastVoiceMsgId;
    long long _lastTextMsgId;
    
    BOOL _isFirst;
    UITapGestureRecognizer *_tagGes;
    
    NSInteger _onlinePageNo;
    NSArray *_onMicroArray; //保存上麦的位置
    
    NSMutableDictionary *_callUserDic;//要@的用户字典
}
@end

@implementation ChatPanelViewController

- (void)dealloc
{
    YJ_RELEASE(_chatbarInfo);
    YJ_RELEASE(_voiceView);
    YJ_RELEASE(_inputView);
    
    [_facade removeNotification:self];
}

- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    if (_chatbarInfo != nil) {
        //显示聊吧浮动窗口
        [_facade postNotification:NOTIF_COMMON_SHOW_CHATBAR_VIEW
                         userInfo:[NSDictionary dictionaryWithObjectsAndKeys:KEY_CHATBAR_INFO,_chatbarInfo, nil]];
        [_facade getSummaryInfo].curChatbarInfo = _chatbarInfo;
        [_facade getSummaryInfo].chatPanelView = self;
    }
}

- (id)initWithChatbarInfo:(ChatbarInfo *)chatbarInfo
{
    self = [super init];
    if (self)
    {
        _chatbarInfo = chatbarInfo;
        _chatbarProxy = [_facade getChatbarProxy];
        if (chatbarInfo.chatbarId != [_facade getSummaryInfo].curChatbarInfo.chatbarId)
        {
            [_chatbarProxy leaveChatbarRoomWithChatbarInfo:_chatbarInfo];
            [_chatbarProxy enterChatbar:_chatbarInfo];
            [_facade getSummaryInfo].curChatbarInfo = chatbarInfo;
        }
        
        [_facade getSummaryInfo].curChatbarInfo = chatbarInfo;
        
        
        
        
        
        // 移除聊吧浮层，进入聊吧时，如果有聊吧浮层显示就移除掉
        [[UIApplication sharedApplication].keyWindow.subviews enumerateObjectsUsingBlock:^(__kindof UIView * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            // 通过浮层对应的tag和浮层类型来判断并移除
            if ([obj isKindOfClass:[UIImageView class]] && 201 == obj.tag) {
                    [obj removeFromSuperview];
            }
        }];
        
        _curUserProxy = [_facade getCurrentUser];
        _keywordProxy = [_facade getKeywordProxy];
        
        _isFirst = YES;
    }
    return self;
}

#pragma mark - 设置导航栏默认背景
- (void)setNavigationBarBackgroundImage
{
    UIImage *img;
    if(IOS7_OR_LATER) // iOS7之前显示这个
    {
        img = [UIImage imageNamed:@"userinfo_navBg128"];
    }
    else
    {
        img = [UIImage imageNamed:@"userinfo_navBg88"];
    }
    
    UIEdgeInsets edge = UIEdgeInsetsMake(0, 0, 0, 0);
    img = [img resizableImageWithCapInsets:edge resizingMode:UIImageResizingModeTile];
    [self.navigationController.navigationBar setBackgroundImage:img
                                                  forBarMetrics:UIBarMetricsDefault];
}

- (void)initViews
{
    [super initViews];
    
    // 设置导航栏背景
    [self setNavigationBarBackgroundImage];
    
    _rightButton = [UIButton btnWithImage:UIIMAGE(@"group_info_btn")
                           imageHighlight:nil
                                    bgImg:nil
                           bgImgHighlight:nil
                                   target:self
                                 clickSel:@selector(pushtoChatbarInfo)];
    self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc] initWithCustomView:_rightButton];
    
    _titleView = [[YJTitleView alloc] init];
    _titleView.delegate = self;
    _titleView.userInteractionEnabled = YES;
    self.navigationItem.titleView = _titleView;
    [self setTitleView];
    
    _voiceView = [[ChatPanelVoiceMessageView alloc] initWithFrame:CGRectMake(0, 0, self.view.width, self.view.height*0.4)
                                                      chatbarInfo:_chatbarInfo
                                                andViewController:self];
    _voiceView.delegate = self;
    [self.view addSubview:_voiceView];
    
    _publicView = [[ChatPanelPublicMessageView alloc] initWithFrame:CGRectMake(0, _voiceView.buttom, self.view.width, self.view.height*0.6)
                                                    withChatbarInfo:_chatbarInfo
                                                  andViewController:self];
    _publicView.delegate = self;
    [self.view addSubview:_publicView];
    
    _inputView = [[YJChatPanelInputView alloc] initWithFrame:CGRectMake(0, 0, self.view.width, 0) type:YJKeyboardTypeSystem];
    _inputView.chatbarInfo = _chatbarInfo;
    _inputView.iMaxContentLength = 140;
    _inputView.inputViewDelegate = self;
    [_inputView showInView:self.view];
    
    //进入房间成功，开始请求数据
    [_facade addNotification:NOTIF_BLL_DEAL_REQUEST_CHATPANEL_DATA
                      target:self
                    selector:@selector(beginLoadData:)];
    [_facade addNotification:NOTIF_BLL_DEAL_ENTER_CHATPANEL_FAI
                      target:self
                    selector:@selector(dealEnterChatPanelFai:)];
    [_facade addNotification:NOTIF_BLL_DEAL_REQUEST_CHATPANEL_DATA_SUC
                      target:self
                    selector:@selector(requestChatpanelDataSuc:)];
    [_facade addNotification:NOTIF_BLL_DEAL_REQUEST_CHATPANEL_DATA_FAI
                      target:self
                    selector:@selector(requestChatpanelDataFai:)];
    
    [_facade addNotification:NOTIF_COMMON_HIDE_KEYBOARD_VIEW
                      target:self
                    selector:@selector(hideKeyboardView:)];
    
//    [_facade addNotification:NOTIF_BLL_SEND_VOICE_MESSAGE_SUC
//                      target:self
//                    selector:@selector(sendVoiceMessageSuc:)];
    
    [_facade addNotification:NOTIF_BLL_PUSH_CHATBAR_NEW_MESSAGE
                      target:self
                    selector:@selector(pushChatbarMessage:)];
    
    [_facade addNotification:NOTIF_COMMON_GET_CHATPANEL_ONLINELIST_RESULT
                      target:self
                    selector:@selector(getChatPanelOnlineListSuc:)];
    
    [_facade addNotification:NOTIF_COMMON_GET_CHATPANEL_CONTRIBUTE_LIST_RESULT
                      target:self
                    selector:@selector(getChatPanelContributeListSuc:)];
    
    [_facade addNotification:NOTIF_COMMON_ATTENTION_CHATBAR_RESULT
                      target:self
                    selector:@selector(attentionChatbarSuc:)];
    
    [_facade addNotification:NOTIF_COMMON_JOINT_CHATBAR_RESULT
                      target:self
                    selector:@selector(joinChatbarSuc:)];
    
    [_facade addNotification:NOTIF_BLL_PUSH_CHATBAR_ONLINE_COUNT
                      target:self
                    selector:@selector(updateChatbarOnlineCount:)];
    
    [_facade addNotification:NOTIF_BLL_PUSH_WHEAT_POSITION
                      target:self
                    selector:@selector(getChatbarWheatPosition:)];
    
    [_facade addNotification:NOTIF_COMMON_SEND_SKILLS_RESULT
                      target:self
                    selector:@selector(sendSkillsMessage:)];
    
    [_facade addNotification:NOTIF_COMMON_SEND_GIFTS_RESULT
                      target:self
                    selector:@selector(sendGiftsMessage:)];
    
    [_facade addNotification:NOTIF_BLL_PUSH_CHATBAR_OPERATION
                      target:self
                    selector:@selector(pushChatbarOperation:)];
    
    [_facade addNotification:@"NOTIF_COMMON_SKILL_MERCY"
                      target:self
                    selector:@selector(sendSkillMercy:)];
    
    _publicMessageArr = [[NSMutableArray alloc] initWithCapacity:0];
    _voiceMessageArr = [[NSMutableArray alloc] initWithCapacity:0];
    _lastVoiceMsgId = 0;
    _lastTextMsgId = 0;
    
    _onlinePageNo = 1;
    _onMicroArray = [[NSArray alloc] init];
    
    _callUserDic = [[NSMutableDictionary alloc] init];
    [YJEmoji setGroupCallUserDic:_callUserDic];
    
    //获取礼物列表
    [_chatbarProxy getChatPanelGiftsList];
    //获取技能列表
    [_chatbarProxy getChatPanelSkillsList];
    
    // 聊吧面板广播视图
    [self initChatbarBroadcastView];
}


#pragma mark - 设置聊吧广播视图
/** 初始化聊吧广播视图 */
- (void)initChatbarBroadcastView
{
     _chatbarBroadcastView = [[ChatbarBroadcastView alloc] initWithFrame:CGRectMake(0, _voiceView.height - 52 - 24, YJ_WIDTH, 24)];
    [_chatbarBroadcastView setAutoScrollWithType:ChatbarBroadcastViewTypeChatpanel];
    _chatbarBroadcastView.delegate = self;
    [_voiceView addSubview:_chatbarBroadcastView];
    
    
    if ([[U6AppFacade getInstance] getSummaryInfo].chatbarAdArr.count <= 0)
    {
        // 不显示广播视图
        _chatbarBroadcastView.hidden = YES;
    }
    else
    {
        _chatbarBroadcastView.hidden = NO;
    }
}


#pragma mark 聊吧广播视图代理：ChatbarBroadcastViewDelegate
- (void)ChatbarBroadcastViewDidClickUserButtonWithUserid:(long long)userid
{
    UserInfoViewController *vc = [[UserInfoViewController alloc] initWithUserId:userid];
    [self.navigationController pushViewController:vc animated:YES];
}
- (void)ChatbarBroadcastViewDidClickChatbatButtonWithChatInfo:(ChatbarInfo *)chatInfo
{
    ChatPanelViewController *vc = [[ChatPanelViewController alloc] initWithChatbarInfo:chatInfo];
    [self.navigationController pushViewController:vc animated:YES];
}
#pragma mark - YJTitleViewDelegate代理方法
/** 子标题点击跳转到在线成员列表 */
- (void)titleView:(YJTitleView *)titleView didClickStateTitle:(BOOL)isClicked
{
    // 跳转到成员列表
    ChatbarInfoMemberViewController *vc = [[ChatbarInfoMemberViewController alloc] initWithChatbarInfo:_chatbarInfo];
    [self.navigationController pushViewController:vc animated:YES];
}

#pragma mark -
- (void)leftBarButtonClick
{
    
//    [_facade getSummaryInfo].chatPanelView = self;
//    // 不管弹出多少聊吧面板控制器，只要点击返回按钮就回到入口控制器
//    UIViewController *enterVC = [[UIViewController alloc] init]; // 入口控制器
//    NSInteger vcCount = self.navigationController.viewControllers.count;
//    for (NSInteger i = 0 ; i < vcCount; i ++ ) {
//        UIViewController *vc = self.navigationController.viewControllers[i];
//        
//        // 寻找第一个聊吧面板，如果找到就跳转到上一个控制器
//        if ([vc isKindOfClass:[ChatPanelViewController class]]) {
//            // 寻找上一个控制器
//            enterVC = self.navigationController.viewControllers[i - 1];
//            [self.navigationController popToViewController:enterVC animated:YES];
//            break;
//        }
//    }
    
//    //显示聊吧浮动窗口
//    [_facade postNotification:NOTIF_COMMON_SHOW_CHATBAR_VIEW
//                     userInfo:[NSDictionary dictionaryWithObjectsAndKeys:KEY_CHATBAR_INFO,_chatbarInfo, nil]];
    [super leftBarButtonClick];
}

- (void)pushtoChatbarInfo
{
    ChatbarInfoViewController *vc = [[ChatbarInfoViewController alloc] initWithChatbarInfo:_chatbarInfo];
    [self.navigationController pushViewController:vc animated:YES];
}


#pragma mark - 设置吧昵称，在线数
- (void)setTitleView
{
    [_titleView setTitle:_chatbarInfo.titleStr];
    NSString *onlineStr = [NSString stringWithFormat:NSLocalizedString(@"CHATPANEL_TITLE_ONLINE", @"(%d在线)"),_chatbarInfo.onlineCount];
    [_titleView setOtherTitle:onlineStr];
}

#pragma mark - YJChatPanelInputViewDelegate
- (void)chatInputView:(YJChatPanelInputView *)chatInputView
     confirmSendSound:(NSString *)fileName
             sendFlag:(NSString *)sendFlag
                 time:(NSInteger)iSec
{
    if ([fileName length] > 0)
    {
        //发送录音消息
        MessageInfo *messageInfo = [MessageInfo createAudioMessage:fileName time:iSec sendUserInfo:_curUserProxy.info autoUploadAttachemnt:NO];
        messageInfo.messageState = MessageStateSubmit;
        messageInfo.sendFlagString = sendFlag;
        messageInfo.mediaMessage.flag = sendFlag;
        [_voiceMessageArr addObject:messageInfo];
        [_voiceView setMessageInfoArr:_voiceMessageArr];
        
        [[YJAudioManager instance] playSendMsgSound];
    }
}

- (void)chatInputView:(YJChatPanelInputView *)chatInputView sendText:(NSString *)text
{
    if (![(MeYouAppDelegate *)[UIApplication sharedApplication].delegate checkUserState])
    {
        return;
    }
    
    text = [text stringByTrimmingCharactersInSet:[NSCharacterSet newlineCharacterSet]];
    if (![text isKindOfClass:[NSString class]] || [text isEqualToString:@""])
    {
        BlockAlertView *alertView = [[BlockAlertView alloc] initWithTitle:NSLocalizedString(@"COMMON_ALTER_TITLE",@"")
                                                                  message:NSLocalizedString(@"MESSAGE_INFO_CONTENT_IS_EMPTY", @"内容不能为空")];
        [alertView setCancelButtonWithTitle:NSLocalizedString(@"INFO_BUTTON_KNOW", @"知道了") block:nil];
        [alertView show];
        return;
    }
    
    // 过滤含有敏感字消息
    NSString* contentString = [_keywordProxy replaceKeywordsForContent:text];
    contentString = [U6Common filterContent:contentString];
    
    NSString *content = [YJEmoji subStringOfEmojiContent:contentString withLength:1000];
    MessageInfo *info = [[MessageInfo alloc] init];
    info.datetime = [[NSDate date] timeIntervalSince1970] * 1000;
    info.user = _curUserProxy.info;
    info.content = content;
    info.attachment = nil;
    info.type = MessageTypeText;
    
    //提交数据
    NSMutableArray *userIds = [NSMutableArray arrayWithArray:[_callUserDic allKeys]];
    for (NSString *value in userIds)
    {
        NSString *nameStr = [_callUserDic objectForKey:value];
        if ([content rangeOfString:nameStr].location == NSNotFound)//(![content containsString:nameStr])
        {
            [userIds removeObject:value];
            [_callUserDic removeObjectForKey:value];
        }
        else
        {
            [info.userNameArray addObject:nameStr];
        }
    }
    [info.userIdArray addObjectsFromArray:userIds];
    
    [_publicMessageArr addObject:info];
    [_publicView setMessageListArray:_publicMessageArr];
    
    [_chatbarProxy sendChatbarMessage:info chatbarInfo:_chatbarInfo callUserList:_callUserDic];
    [_inputView killFocus];
    [_callUserDic removeAllObjects];
}

- (void)chatInputViewWillShowKeyboard:(YJChatPanelInputView *)chatInputView keyboardType:(YJKeyboardType)keyboardType durationTime:(double)time
{
    _publicView.isNeedShowInputView = YES;
    [UIView beginAnimations:@"changeListFrame" context:nil];
    [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
    [UIView setAnimationDuration:time];
    
    _voiceView.frame = CGRectMake(0, -(chatInputView.height-44), _voiceView.width, _voiceView.height);
    _publicView.frame = CGRectMake(0, _voiceView.buttom, _publicView.width, _publicView.height);
    
    [UIView commitAnimations];
}

- (void)chatInputViewWillHideKeyboard:(YJChatPanelInputView *)chatInputView
{
    _publicView.isNeedShowInputView = NO;
    [UIView beginAnimations:@"changeListFrame" context:nil];
    [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
    [UIView setAnimationDuration:0.3];
    
    _voiceView.frame = CGRectMake(0, 0, _voiceView.width, _voiceView.height);
    _publicView.frame = CGRectMake(0, _voiceView.buttom, _publicView.width, _publicView.height);
    
    [UIView commitAnimations];
}

- (void)chatInputViewOnclickSendGifts:(YJChatPanelInputView *)chatInputView
{
    NSMutableArray *userInfoArray = [[NSMutableArray alloc] init];
    for (int i = 0; i < _onMicroArray.count; i++)
    {
        id info = [_onMicroArray objectAtIndex:i];
        if ([info isKindOfClass:[ChatbarInfo class]])
        {
            ChatbarInfo *chatbarInfo = [_onMicroArray objectAtIndex:i];
            if (chatbarInfo.userInfo != nil) {
                [userInfoArray addObject:chatbarInfo.userInfo];
            }
        }
    }
    //点击快捷送礼
    ChatPanelGiftSheetView *giftSheet = [[ChatPanelGiftSheetView alloc] initWithGiftsArray:[_facade getSummaryInfo].giftsArray andUserArray:userInfoArray];
    giftSheet.delegate = self;
    [giftSheet showInView];
}

#pragma mark - ChatPanelPublicMessageViewDelegate
- (void)hideChatPanelInputView
{
    [_inputView killFocus];
}

- (void)refreshChatPanelMessageData
{
    [_chatbarProxy getChatbarHistoryMsgListWithChatbarId:_chatbarInfo.chatbarId
                                                    type:1
                                                maxMsgId:_lastTextMsgId
                                                  amount:50];
}

- (void)removeChatPanelInputView:(BOOL)isRemove
{
    if (isRemove)
    {
        [_inputView hide];
    }
    else
    {
        [_inputView showInView:self.view];
    }
}

- (void)deleteChatPanel:(BOOL)isDelete
{
    _inputView.userInteractionEnabled = !isDelete;
    [_inputView killFocus];
}

- (void)onlongClickWithUserInfo:(UserInfo *)userInfo
{
    [self longClickOnAvatarView:userInfo];
}

#pragma mark - ChatPanelVoiceMessageViewDelegate
- (void)clickonRequestWheat:(NSInteger)position withOperateType:(NSInteger)type
{
    [_chatbarProxy operateChatbarWheatWithChatbarId:_chatbarInfo.chatbarId
                                             userId:[_facade getCurrentUser].info.userId
                                           position:position
                                               type:type];
}

- (void)attentionOnChatbarAction:(NSInteger)operateType
{
    //加入
    if (operateType == 0)
    {
        [_chatbarProxy joinChatbarWithChatbarId:_chatbarInfo.chatbarId];
    }
    else if (operateType == 2)//关注
    {
        [_chatbarProxy attentionChatbarWithChatbarId:_chatbarInfo.chatbarId];
    }
}

- (void)refreshChatPanelVoiceMessageData
{
    [_chatbarProxy getChatbarHistoryMsgListWithChatbarId:_chatbarInfo.chatbarId
                                                    type:3
                                                maxMsgId:_lastVoiceMsgId
                                                  amount:10];
}

- (void)longClickOnAvatarView:(UserInfo *)userInfo
{
    if (userInfo.userId != _curUserProxy.info.userId)
    {
        NSString *keyStr = [NSString stringWithFormat:@"%lld", userInfo.userId];
        NSString *str = [NSString stringWithFormat:
                         @"@%@ ",
                         userInfo.nickName];
        if ([[_callUserDic allKeys] containsObject:keyStr] &&
            [_inputView.contentString rangeOfString:str].location != NSNotFound)
        {
            return;
        }
        
        [_inputView becomeInputMode];
        
        if ([str length] > _inputView.iMaxContentLength)
        {
            str = [YJEmoji subStringOfEmojiContent:str withLength:_inputView.iMaxContentLength];
        }
        
        [_inputView insertTextAfterCursor:str];
        [_inputView setFocus];
        
        NSString *userIdStr = [NSString stringWithFormat:@"%lld", userInfo.userId];
        [_callUserDic setObject:str forKey:userIdStr];
    }
}

#pragma mark - Chat
- (void)sendGiftInfo:(GiftInfo *)giftInfo toUserInfo:(UserInfo *)userInfo withGiftNum:(NSInteger)giftNum
{
    [_chatbarProxy sendGiftsToChatbarId:_chatbarInfo.chatbarId
                               userInfo:userInfo
                                 giftid:giftInfo.iGiftid
                                paytype:giftInfo.iCurrencyType
                                giftnum:giftNum];
}

#pragma mark - NSNotification
- (void)beginLoadData:(NSNotification *)notif
{
    NSDictionary *dic = notif.userInfo;
    _chatbarInfo.titleStr = [dic objectForKey:@"chatbarname"];
    _chatbarInfo.fanscount = [[dic objectForKey:@"fanscount"] longValue];
    _chatbarInfo.membercount = [[dic objectForKey:@"membercount"] integerValue];
    _chatbarInfo.onlineCount = [[dic objectForKey:@"onlinecount"] integerValue];
    [_facade getCurrentUser].info.chatbarRole = [[dic objectForKey:@"role"] integerValue];
    [_facade getCurrentUser].info.chatbarJoinstatus = [[dic objectForKey:@"joinstatus"] integerValue];
    
    if ([_facade getCurrentUser].info.chatbarRole != 5)
    {
        [_voiceView updateAttention:_chatbarInfo.fanscount];
    }
    else
    {
        if([_facade getCurrentUser].info.chatbarJoinstatus == 1 || [_facade getCurrentUser].info.chatbarJoinstatus == 3) //已关注,已加入
        {
            [_voiceView updateAttention:_chatbarInfo.fanscount];
        }
    }
    
    NSArray *onMicroArray = [dic objectForKey:@"microphonelist"];
    NSMutableArray *microPlaceArr = [[NSMutableArray alloc] init];
    if ([onMicroArray count] > 0)
    {
        for (int i = 0; i < onMicroArray.count; i++)
        {
            ChatbarInfo *info = [[ChatbarInfo alloc] init];
            [info importFromChatPanelServerData:onMicroArray[i]];
            [microPlaceArr addObject:info];
        }
        [_voiceView updateWheatPosition:microPlaceArr];
        _onMicroArray = microPlaceArr;
        
        [[_facade getSummaryInfo].onMicroArray removeAllObjects];
        [[_facade getSummaryInfo].onMicroArray addObjectsFromArray:microPlaceArr];
    }
    
    [self setTitleView];
    
    //吧主或者是管理员进入房间后自动上麦
    if ([_facade getCurrentUser].info.chatbarRole == 0 || [_facade getCurrentUser].info.chatbarRole == 1)
    {
        [self authorOperationMicro];
    }
    
    [_publicView updateOnlineCount:[[dic objectForKey:@"onlinecount"] integerValue]];
    
    [_chatbarProxy getChatbarHistoryMsgListWithChatbarId:_chatbarInfo.chatbarId
                                                    type:3
                                                maxMsgId:0
                                                  amount:5];
    [_chatbarProxy getChatbarHistoryMsgListWithChatbarId:_chatbarInfo.chatbarId
                                                    type:1
                                                maxMsgId:0
                                                  amount:50];
    
    //更改输入方式
    BOOL isOpenSound = YES;
    for (ChatbarInfo *info in microPlaceArr)
    {
        if (info.userInfo.userId == [_facade getCurrentUser].info.userId)
        {
            [_inputView setSoundAction:YES];
            isOpenSound = NO;
            break;
        }
    }
    
    if (isOpenSound == YES)
    {
        [_inputView setSoundAction:NO];
    }
}

//自动上麦
NSComparator chatbarPanelViewComparator = ^(id obj1, id obj2)
{
    if ([obj1 integerValue] > [obj2 integerValue])
    {
        return (NSComparisonResult)NSOrderedDescending;
    }
    
    if ([obj1 integerValue] < [obj2 integerValue])
    {
        return (NSComparisonResult)NSOrderedAscending;
    }
    return (NSComparisonResult)NSOrderedSame;
};

- (void)authorOperationMicro
{
    NSMutableArray *positionArr = [[NSMutableArray alloc] init];
    for (NSInteger i = 0; i < [_facade getSummaryInfo].onMicroArray.count; i++)
    {
        ChatbarInfo *info = [[_facade getSummaryInfo].onMicroArray objectAtIndex:i];
        if (info.userInfo.userId == [_facade getCurrentUser].info.userId)
        {
            return;
        }
        [positionArr addObject:[NSNumber numberWithInteger:info.position]];
    }
    
    NSInteger index = 1;
    if (positionArr.count > 0)
    {
        [positionArr sortUsingComparator:chatbarPanelViewComparator];
        
        for (NSInteger i = 0; i < positionArr.count; i++)
        {
            if (index != [[positionArr objectAtIndex:i] integerValue])
            {
                break;
            }
            index++;
        }
    }
    
    if (index >= 6)
    {
        return;
    }
    
    [[_facade getChatbarProxy] operateChatbarWheatWithChatbarId:_chatbarInfo.chatbarId
                                                         userId:[_facade getCurrentUser].info.userId
                                                       position:index
                                                           type:0];
}

- (void)requestChatpanelDataSuc:(NSNotification *)notif
{
    NSArray *messageArray = [notif.userInfo objectForKey:KEY_MESSAGE_LIST];
    NSInteger type = [[notif.userInfo objectForKey:KEY_MESSAGE_TYPE] integerValue];
    if (type == 3)
    {
        [_voiceMessageArr insertObjects:messageArray atIndexes:[NSIndexSet indexSetWithIndexesInRange:NSMakeRange(0, messageArray.count)]];
        [_voiceView setMessageInfoArr:_voiceMessageArr];
        
        if (_voiceMessageArr.count > 0)
        {
            MessageInfo *info = [_voiceMessageArr objectAtIndex:0];
            _lastVoiceMsgId = info.llMsgId;
        }
    }
    else
    {
        [_publicMessageArr insertObjects:messageArray atIndexes:[NSIndexSet indexSetWithIndexesInRange:NSMakeRange(0, messageArray.count)]];
        [_publicView setMessageListArray:_publicMessageArr];
        
        if (_publicMessageArr.count > 0)
        {
            MessageInfo *info = [_publicMessageArr objectAtIndex:0];
            _lastTextMsgId = info.llMsgId;
        }
    }
}

- (void)requestChatpanelDataFai:(NSNotification *)notif
{
    if ([[notif userInfo] objectForKey:KEY_ERROR_CODE])
    {
        [self presentFailureTips:[[notif userInfo] objectForKey:KEY_ERROR_TEXT]];
    }
    else
    {
        [self presentFailureTips:NSLocalizedString(@"MESSAGE_NET_ERROR", @" 您的网络好像有点问题，请检查一下哦！")];
    }
}

- (void)dealEnterChatPanelFai:(NSNotification *)notif
{
    if ([[notif userInfo] objectForKey:KEY_ERROR_CODE])
    {
        [self presentFailureTips:[[notif userInfo] objectForKey:KEY_ERROR_TEXT]];
    }
    else
    {
        [self presentFailureTips:NSLocalizedString(@"MESSAGE_NET_ERROR", @" 您的网络好像有点问题，请检查一下哦！")];
    }
}

#pragma mark - 接受新消息
/** 接受新消息 */
- (void)pushChatbarMessage:(NSNotification *)notif
{
    long long chatbarId = [[notif.userInfo objectForKey:KEY_CHATBAR_ID] longLongValue];
    if (chatbarId == _chatbarInfo.chatbarId)
    {
        MessageInfo *info = [notif.userInfo objectForKey:KEY_MESSAGE_INFO];
        if (info.type == 3) // 语音区，语音
        {
            [_voiceMessageArr addObject:info];
            [_voiceView setMessageInfoArr:_voiceMessageArr];
            // 显示更多消息按钮
//            [_voiceView showButtonWithType:ChatPanelVoiceMessageViewButtonTypeNewMessageBtn];
        }
        else // 文字，公聊区
        {
            [_publicMessageArr addObject:info];
            [_publicView setMessageListArray:_publicMessageArr];
        }
    }
}



#pragma mark - 获取聊天面板的在线用户
- (void)getChatPanelOnlineList
{
    [_chatbarProxy getChatbarOnlineListWithChatbarId:_chatbarInfo.chatbarId
                                              pageNo:_onlinePageNo
                                            pageSize:15];
}

- (void)getChatPanelContributeList
{
    [_chatbarProxy getChatbarContributeListWithChatbarId:_chatbarInfo.chatbarId
                                                  pageNo:1
                                                pageSize:20];
}

- (void)getChatPanelOnlineListSuc:(NSNotification *)notif
{
    NSArray *onlineArr = [notif.userInfo objectForKey:KEY_CHATBAR_ONLINE_LIST];
    [_publicView updateOnlineList:onlineArr andIsMore:YES];
}

- (void)getChatPanelContributeListSuc:(NSNotification *)notif
{
    NSArray *userInfoArray = [notif.userInfo objectForKey:KEY_CHATBAR_CONTRIBUTE_LIST];
    [_publicView updateContributeList:userInfoArray];
}

- (void)attentionChatbarSuc:(NSNotification *)notif
{
    long attentionCount = [[notif.userInfo objectForKey:@"attentions"] longValue];
    [_voiceView updateAttention:attentionCount];
}

- (void)joinChatbarSuc:(NSNotification *)notif
{
    
}

#pragma mark - 当键盘弹起的时候切换到后台，后台回来后的处理
- (void)hideKeyboardView:(NSNotification *)notif
{
    [_inputView killFocus];
}

#pragma mark - 推送在线用户数
- (void)updateChatbarOnlineCount:(NSNotification *)notif
{
    long long chatbarId = [[notif.userInfo objectForKey:@"chatbarid"] longLongValue];
    NSInteger onlineCount = [[notif.userInfo objectForKey:@"onlinecount"] integerValue];
    if (chatbarId == _chatbarInfo.chatbarId)
    {
        _chatbarInfo.onlineCount = onlineCount;
        [self setTitleView];
        [_publicView updateOnlineCount:onlineCount];
    }
}

#pragma mark - 推送麦克风位置，刷新麦克风位置
- (void)getChatbarWheatPosition:(NSNotification *)notif
{
    long long chatbarId = [[notif.userInfo objectForKey:KEY_CHATBAR_ID] longLongValue];
    if (chatbarId == _chatbarInfo.chatbarId)
    {
        NSArray *positionArr = [notif.userInfo objectForKey:KEY_CHATBAR_ONMICRO_POSITION];
        [_voiceView updateWheatPosition:positionArr];
        
        _onMicroArray = positionArr;
        
        BOOL isOpenSound = YES;
        for (ChatbarInfo *info in positionArr)
        {
            if (info.userInfo.userId == [_facade getCurrentUser].info.userId)
            {
                [_inputView setSoundAction:YES];
                isOpenSound = NO;
                break;
            }
        }
        
        if (isOpenSound == YES)
        {
            [_inputView setSoundAction:NO];
        }
    }
}

#pragma mark - 施放技能成功后发送消息
- (void)sendSkillsMessage:(NSNotification *)notif
{
    MessageInfo *info = [notif.userInfo objectForKey:@"messageinfo"];
    NSMutableDictionary *dic = [[NSMutableDictionary alloc] init];
    if (info.userIdArray.count > 0)
    {
        for (int i = 0; i < info.userIdArray.count; i++)
        {
            NSString *userIdStr = [NSString stringWithFormat:@"%lld",[[info.userIdArray objectAtIndex:i] longLongValue]];
            NSString *str = [NSString stringWithFormat:@"%@",[info.userNameArray objectAtIndex:i]];
            [dic setObject:str forKey:userIdStr];
        }
    }
    [_publicMessageArr addObject:info];
    [_publicView setMessageListArray:_publicMessageArr];
    [_chatbarProxy sendChatbarMessage:info chatbarInfo:_chatbarInfo callUserList:dic];
}

- (void)sendGiftsMessage:(NSNotification *)notif
{
    MessageInfo *info = [notif.userInfo objectForKey:@"messageinfo"];
    NSMutableDictionary *dic = [[NSMutableDictionary alloc] init];
    if (info.userIdArray.count > 0)
    {
        for (int i = 0; i < info.userIdArray.count; i++)
        {
            NSString *userIdStr = [NSString stringWithFormat:@"%lld",[[info.userIdArray objectAtIndex:i] longLongValue]];
            NSString *str = [NSString stringWithFormat:@"%@",[info.userNameArray objectAtIndex:i]];
            [dic setObject:str forKey:userIdStr];
        }
    }
    [_publicMessageArr addObject:info];
    [_publicView setMessageListArray:_publicMessageArr];
    [_chatbarProxy sendChatbarMessage:info chatbarInfo:_chatbarInfo callUserList:dic];
}

//技能求饶
- (void)sendSkillMercy:(NSNotification *)notif
{
    MessageInfo *info = [notif.userInfo objectForKey:@"messageInfo"];
    [_publicMessageArr addObject:info];
    [_publicView setMessageListArray:_publicMessageArr];
}

#pragma mark - 推送管理员等操作
- (void)pushChatbarOperation:(NSNotification *)notif
{
    long long chatbarId = [[notif.userInfo objectForKey:@"chatbarid"] longLongValue];
    if (chatbarId == _chatbarInfo.chatbarId)
    {
        NSInteger type = [[notif.userInfo objectForKey:@"type"] integerValue];
        long long userid = [[notif.userInfo objectForKey:@"userid"] longLongValue];
//        long long expiredtime = [[notif.userInfo objectForKey:@"expiredtime"] longLongValue];
        switch (type)
        {
            case 0://踢出
            {
                [_chatbarProxy leaveChatbarRoomWithChatbarInfo:_chatbarInfo];
                [self.navigationController popViewControllerAnimated:YES];
            }
                break;
            case 1://禁言5分钟
            {
                if (userid == [_facade getCurrentUser].info.userId)
                {
                    [self presentFailureTips:NSLocalizedString(@"CHATPANEL_BANNED", @"你已被禁言5分钟")];
                }
            }
                break;
            case 2://解除禁言
            {
                if (userid == [_facade getCurrentUser].info.userId)
                {
                    [self presentFailureTips:NSLocalizedString(@"CHATPANEL_UNBANNED", @"你被解除禁言")];
                }
            }
                break;
                
            case 3://邀请加入
                break;
                
            case 4://封号5分钟
            {
                [_chatbarProxy leaveChatbarRoomWithChatbarInfo:_chatbarInfo];
                [self.navigationController popViewControllerAnimated:YES];
            }
                break;
                
            case 5://解除封号
                break;
                
            case 6://解除管理员
            {
                if (userid == [_facade getCurrentUser].info.userId)
                {
                    [self presentFailureTips:NSLocalizedString(@"CHATPANEL_CLOSE_ADMIN", @"你被解除管理员")];
                }
            }
                break;
            case 7://解除禁言
            {
                if (userid == [_facade getCurrentUser].info.userId)
                {
                    [self presentFailureTips:NSLocalizedString(@"CHATPANEL_BECOME_ADMIN",@"你被升为管理员")];
                }
            }
                break;
                
            default:
                break;
        }
    }
}

@end
