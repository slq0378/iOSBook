# UIView相关

typedef enum{
	EGOOPullRefreshPulling = 0,
	EGOOPullRefreshNormal,
	EGOOPullRefreshLoading,
    EGOOPULLRefreshNoMoreData,
} EGOPullRefreshState;

@protocol EGORefreshTableHeaderDelegate;
@interface EGORefreshTableHeaderView : UIView {
	
	id _delegate;
	EGOPullRefreshState _state;

	RefreshLoadingView *_activityView;
    
    BOOL _bIsEnd;
}

@property(nonatomic,assign) id <EGORefreshTableHeaderDelegate> delegate;
@property  (nonatomic,strong) UILabel *title;
- (void)refreshLastUpdatedDate;
- (void)egoRefreshScrollViewDidScroll:(UIScrollView *)scrollView;
- (void)egoRefreshScrollViewDidEndDragging:(UIScrollView *)scrollView;
- (void)egoRefreshScrollViewDataSourceDidFinishedLoading:(UIScrollView *)scrollView;
- (void)setState:(EGOPullRefreshState)aState;

@end
@protocol EGORefreshTableHeaderDelegate
- (void)egoRefreshTableHeaderDidTriggerRefresh:(EGORefreshTableHeaderView*)view;
- (BOOL)egoRefreshTableHeaderDataSourceIsLoading:(EGORefreshTableHeaderView*)view;
@optional
- (NSDate*)egoRefreshTableHeaderDataSourceLastUpdated:(EGORefreshTableHeaderView*)view;
@end



- (id)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
		
		self.autoresizingMask = UIViewAutoresizingFlexibleWidth;
		self.backgroundColor = BACKGROUND_COLOR;
        
		RefreshLoadingView *view = [[RefreshLoadingView alloc]initWithFrame:CGRectMake(0, 0, YJ_WIDTH, ANIMATION_VIEW_SIZE)];
		view.frame = CGRectMake(0, frame.size.height - ANIMATION_VIEW_SIZE - 10, YJ_WIDTH, ANIMATION_VIEW_SIZE);
		[self addSubview:view];
		_activityView = view;
		[view release];
        UILabel *title = [[UILabel alloc] init];
        [self addSubview:title];
        self.title = title;
        [title sizeToFit];
        title.textAlignment = NSTextAlignmentCenter;
        title.frame = view.frame;
        title.hidden = YES;
		[self setState:EGOOPullRefreshNormal];
		
    }
	
    return self;
	
}


- (void)setState:(EGOPullRefreshState)aState{
	
	switch (aState) {
		case EGOOPullRefreshPulling:
			break;
		case EGOOPullRefreshNormal:
			[_activityView stopAnimating];
			break;
		case EGOOPullRefreshLoading:
			[_activityView startAnimating];
			break;
        case EGOOPULLRefreshNoMoreData:
            [_activityView stopAnimating];
            _activityView.hidden = YES;
            self.title.hidden = NO;
            break;
		default:
			break;
	}
	
	_state = aState;
}