# iAround

- 私聊 `PMMessageListViewController`
- 发出搭讪 `AccostSendViewController`
- 侧边栏 `LeftSideViewController`
-

- (void)showSendSkillAnimated:(MessageInfo *)messageInfo
{
    
    if (_skillAnimationViewArray.count == 0)
    {
        [self setSkillAnimationViewWithMessageInfo:messageInfo andHeightRatio:0.4];
    }
    else if (1 == _skillAnimationViewArray.count)
    {
        [self setSkillAnimationViewWithMessageInfo:messageInfo andHeightRatio:0.6];
    }
    else if (_skillAnimationViewArray.count >= 2)
    {
        ChatPanelSkillAminatedView *firstView = (ChatPanelSkillAminatedView *)[_skillAnimationViewArray firstObject];
        ChatPanelSkillAminatedView *secondView = (ChatPanelSkillAminatedView *)[_skillAnimationViewArray lastObject];
        { // 两个
            
            [UIView animateWithDuration:0.2 animations:^{
                [firstView removeFromSuperview]; // 移除动画1
                secondView.frame = firstView.frame;// 动画2上移
            }];
            [_skillAnimationViewArray removeObject:firstView];
            
            [self setSkillAnimationViewWithMessageInfo:messageInfo andHeightRatio:0.6];
        }
    }
}