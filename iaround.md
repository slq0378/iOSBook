# iAround

- 私聊 `PMMessageListViewController`
- 发出搭讪 `AccostSendViewController`
- 侧边栏 `LeftSideViewController`
-





- 广播改动

/// 手动释放定时器
- (void)removeAllTimers
{
    [_scrollTimer1 invalidate];
    [_scrollTimer2 invalidate];
    _scrollTimer2 = nil;
    _scrollTimer1 = nil;
}
- (void)dealloc
{
    [self removeAllTimers];
}


/// 定时器2
- (void)scrollView2
{
    NSInteger count = _scrollView.subviews.count;
    if (count == 0) {
        return;
    }
    ChatbarScrollView *view = (ChatbarScrollView *)_scrollView.subviews[_next]; // 取出View
    CGRect rect = view.frame;
    rect.origin.x -- ;
    view.frame = rect;
    
    if ((view.frame.origin.x <= - view.frame.size.width + YJ_WIDTH - 100) && (view.frame.origin.x >= - view.frame.size.width + YJ_WIDTH - 100 - 3)) {
        // 只有一个广播就不开启下一个定时器
        if (count == 1) {
            _scrollTimer1.paused = YES;
        }else {
            _scrollTimer1.paused = NO;
            // 距离屏幕右侧100时开启下一个定时器
            _index = (_next + 1)%count;
        }
    }
    else if (view.frame.origin.x <= -(view.frame.size.width) && view.frame.origin.x >= -(view.frame.size.width + 3)) {
       // 开启下一个定时器并恢复自身位置
        rect.origin.x  = YJ_WIDTH ;
        view.frame = rect;
        if (count == 1) {
            _scrollTimer2.paused = NO;
        }else {
            _scrollTimer2.paused = YES;
        }
    }
}

/// 定时器1
- (void)scrollView1
{
    NSInteger count = _scrollView.subviews.count;
    if (count == 0) {
        return;
    }
    ChatbarScrollView *view = (ChatbarScrollView *)_scrollView.subviews[_index]; // 取出View
    CGRect rect = view.frame;
    rect.origin.x -- ;
    view.frame = rect;
    
    if ((view.frame.origin.x <= - view.frame.size.width + YJ_WIDTH - 100) && (view.frame.origin.x >= - view.frame.size.width + YJ_WIDTH - 100 - 3)) {
        // 只有一个广播就不开启下一个定时器
        if (count == 1) {
              _scrollTimer2.paused = YES;
        }else {
            _scrollTimer2.paused = NO;
            // 距离屏幕右侧100时开启下一个定时器
            _next = (_index + 1)%count;
        }
    }
    else if (view.frame.origin.x <= -(view.frame.size.width) && view.frame.origin.x >= -(view.frame.size.width + 3)) {
        
        if (count == 1) {
            _scrollTimer1.paused = NO;
        }else {
            _scrollTimer1.paused = YES;
        }
        // 开启下一个定时器并恢复自身位置
        rect.origin.x  = YJ_WIDTH;
        view.frame = rect;

    }
}
