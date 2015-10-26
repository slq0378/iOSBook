# UIScrollView窗口悬停

## UIScrollView使用注意

### UIScrollView内部子控件添加约束的注意点：
 - 子控件的尺寸`不能通过UIScrollView`来计算，可以考虑通过以下方式计算
    - 可以设置`固定值`（width==100，height==300）
    - 可以相对于`UIScrollView以外的其他控件`来计算尺寸
 - UIScrollView的`frame`应该通过`子控件以外的其他控件`来计算
 - UIScrollView的`contentSize`通过子控件来计算
    - 根据`子控件的尺寸`以及`子控件与UIScrollView之间的间距`


## 窗口悬停
###不使用自动布局,使用frame

- 添加几个子控件（UIScrollView -> UIImageView，UIView）,直接设置frame确定尺寸
- 分析
	- 这个UIView在达到窗口顶部时一直显示在窗口顶部，但是UIScrollView可以继续向上滚动
	- 监视UIScrollView的偏移量，当view达到顶部时，将view 从UIScrollView中移除，添加到self.view,向下滑动时，到这个原来view 的位置时再把view添加到UIScrollView，再向下拖动就方法图片
	- 实现时要用一个属性来记录view 在UIScrollView中得位置
	- 注意，这里通过storyboard添加的子控件，没有使用自动布局，所以要把自动布局完毕关闭，不然会出现莫名其妙的问题
- 实现
	- 主要是在滚动scrollView时进行判断

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    // 滚动过程中判断红色view 的位置，如果位于最顶部，就从UIScrollView脱离，添加到self.view，否则就添加到UIScrollView
    // 当前view距离顶部的距离
    CGFloat height = scrollView.contentOffset.y - self.imageView.bounds.size.height;
    if (height >= 0) {
        CGRect temp = _oldRect;
        temp.origin.y = 0;
        self.redView.frame = temp;
        [self.view addSubview:self.redView];
    }
    else
    {
        self.redView.frame = _oldRect;
        [self.blueScrollView addSubview:self.redView];
    }
    // 方法照片
    CGFloat scale = (1 - scrollView.contentOffset.y/70);
    NSLog(@"%f",scale);
    scale = scale >= 1 ? scale : 1;
    self.imageView.transform = CGAffineTransformMakeScale(scale, scale);
}
```
### 自动布局实现

- 自动布局实现貌似又复杂了，使用自动布局的话，不能直接修改frame，否则会出问题,所以这里新建一个大小一致的view，显示到self.view，
- 首先要重新生成一个UIView，最好就是克隆一个和redView一样的view，然后设置在原始位置，滚动到顶部时显示，向下滚动时隐藏。
代码如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];

    _stopView = [[UIView alloc ]init];
    _stopView.frame = self.redView.bounds;
    _stopView.backgroundColor = self.redView.backgroundColor;
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    // 滚动过程中判断红色view 的位置，如果位于最顶部，就从UIScrollView脱离，添加到self.view，否则就添加到UIScrollView
    // 使用自动布局的话，不能直接修改frame，否则会出问题,所以这里新建一个大小一致的view，显示到self.view，
    // 当前view距离顶部的距离
    CGFloat height = self.imageView.bounds.size.height - scrollView.contentOffset.y;

    if (height <= 0) {

        _stopView.hidden = NO;
        [self.view addSubview:_stopView];
    }else
    {
        _stopView.hidden = YES;
        [self.blueScrollView addSubview:self.redView];
    }

    // 缩放图片
    CGFloat scale = (1 - scrollView.contentOffset.y/70);

    scale = scale >= 1 ? scale : 1;
    self.imageView.transform = CGAffineTransformMakeScale(scale, scale);
}
```





