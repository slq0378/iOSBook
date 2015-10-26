
# iPad开发

- 点分辨率：768*1024
- 旋转方向：iPad4个，iPhone3个
- iPad特有API
	- UIPopoverCpntroller

![](/images/屏幕快照 2015-08-09 09.11.29.png)
	- UISplitViewController

![](/images/屏幕快照 2015-08-09 09.12.26.png)

- 共有API差异
	- UIActionSheet

![](/images/屏幕快照 2015-08-09 09.14.03.png)

- UIButon 中image和title调整
 `-(void)imageRectForContentRect:(CGRect)contentRect;`
 `-(void)titleRectForContentRect:(CGRect)contentRect;`

- Modal
	- iphone - 全屏从下弹出
	- iPad - 4种样式:Modal出来的控制器，最终显示出来的样子
	`modalPresentationStyle`
		 - UIModalPresentationFullScreen (全屏)
		 - UIModalPresentationPageSheet（填充，显示状态栏）
		 - UIModalPresentationFormSheet（显示一个窗口，一部分）
		 - UIModalPresentationCurrentContext（和父控件一致）
	- Modal一共4种过渡样式:Modal出来的控制器，是以怎样的动画呈现出来
		- UIModalTransitionStyleCoverVertical ：从底部往上钻（默认）
		- UIModalTransitionStyleFlipHorizontal ：三维翻转
		- UIModalTransitionStyleCrossDissolve ：淡入淡出
		- UIModalTransitionStylePartialCurl ：翻页（只显示部分，使用前提：呈现样式必须是UIModalPresentationFullScreen）

- 监听屏幕的旋转或父控件的旋转

```objc
/**
 *	旋转或者尺寸改变时会调用这个方法
 */
- (void)viewWillTransitionToSize:(CGSize)size withTransitionCoordinator:(id<UIViewControllerTransitionCoordinator>)coordinator
{
    [super viewWillTransitionToSize:size withTransitionCoordinator:coordinator];

    // 宽度随旋转方向改变
    BOOL isLandscape = size.width == kLandscapeWidth;
    // 获取旋转动动画时间
    CGFloat dura = [coordinator transitionDuration];
    [UIView animateWithDuration:dura animations:^{

        [self.dockView rotateByOrientation:isLandscape];
    }];
}
```

### 2015-8-10
- 上午（11：30之前视频快速浏览一遍）
- `UIPopoverCpntroller`
- 顶部自定义控件：默认xibframe随父控件frame改变而改变，修改xib的autoresizing
- 给控件增加addTarget方法，传递方法
- UIPopoverCpntroller
	- 方法1：
```objc
    // 1、指定ContentViewController，必须在创建时指定这个contentController
    SLQMenuViewController *menuVC = [[SLQMenuViewController alloc] init];
    // 弹出UIPopocerViewController
    UIPopoverController *pop = [[UIPopoverController alloc] initWithContentViewController:menuVC];
    // 2、设置尺寸，可以在menuVC指定尺寸 preferredContentSize
//    pop.popoverContentSize = CGSizeMake(100, 300);
    // 3、设置在什么地方显示
    [pop presentPopoverFromBarButtonItem:self.leftItem permittedArrowDirections:UIPopoverArrowDirectionAny animated:YES];
```

- iOS8 popover使用
- UIPopoverController类型的属性使用strong。主要是因为没有其他强引用指向这个属性，所以为了保持它不死，只能自己使用强引用属性

/** 选择颜色的Popover */
@property (nonatomic, strong) UIPopoverController *colorPopover;

```objc
    // iOS8开始，popover的使用，可以直接通过presentViewController
    SLQTestViewController *testVC = [[SLQTestViewController alloc] init];
    // 设置样式
    testVC.modalPresentationStyle = UIModalPresentationPopover;
    // 设置弹出位置
    testVC.popoverPresentationController.sourceView = self.iOS8Btn;
    testVC.popoverPresentationController.sourceRect = self.iOS8Btn.bounds;
    // 指定barButtonItem为弹出位置
      self.categoryVc.popoverPresentationController.barButtonItem = self.categoryItem;
    [self presentViewController:self.categoryVc animated:YES completion:nil];
    // 弹出
    [self presentViewController:testVC animated:YES completion:nil];
```

- 加载html文件
```objc
    // 路径的话，看情况而定
    NSString *path = [NSString stringWithFormat:@"%@.html",self.food.idstr];
    NSURL *url = [[NSBundle mainBundle] URLForResource:path withExtension:nil];
    [self.webView loadRequest:[NSURLRequest requestWithURL:url]];
```


