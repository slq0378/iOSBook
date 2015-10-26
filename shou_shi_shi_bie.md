
# 手势识别

 ![](/images/手势识别效果.gif)

## UIGestureRecognizer

- 利用UIGestureRecognizer，能轻松识别用户在某个view上面做的一些常见手势
UIGestureRecognizer是一个抽象类，定义了所有手势的基本行为，使用它的子类才能处理具体的手势

```objc
    UITapGestureRecognizer(敲击)
    UIPinchGestureRecognizer(捏合，用于缩放)
    UIPanGestureRecognizer(拖拽)
    UISwipeGestureRecognizer(轻扫)
    UIRotationGestureRecognizer(旋转)
    UILongPressGestureRecognizer(长按)
```

- 使用方法

```objc
    // 创建点按手势
    UITapGestureRecognizer *gesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tap:)];
    // 给imageView添加手势，默认imageView不响应事件
    // 设置imageView 响应事件
    self.imageView.userInteractionEnabled = YES;
    [self.imageView addGestureRecognizer:gesture];
```
- 响应手势事件，可以在里面进行形变

```objc
    // 响应手势事件
    - (void)tap:(UITapGestureRecognizer *)tap
    {
        NSLog(@"%s",__func__);
        // 旋转
        self.imageView.transform = CGAffineTransformRotate(self.imageView.transform, M_PI_4);
    }

```

- 手势的状态

```objc
    typedef NS_ENUM(NSInteger, UIGestureRecognizerState) {
        // 没有触摸事件发生，所有手势识别的默认状态
        UIGestureRecognizerStatePossible,
        // 一个手势已经开始但尚未改变或者完成时
        UIGestureRecognizerStateBegan,
        // 手势状态改变
        UIGestureRecognizerStateChanged,
        // 手势完成
        UIGestureRecognizerStateEnded,
        // 手势取消，恢复至Possible状态
        UIGestureRecognizerStateCancelled,
        // 手势失败，恢复至Possible状态
        UIGestureRecognizerStateFailed,
        // 识别到手势识别
        UIGestureRecognizerStateRecognized = UIGestureRecognizerStateEnded
    };
```
- 手势的一些代理方法

```objc
    #pragma mark -  手势代理方法
    // 是否开始手势
    - (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer
    {
        return YES;
    }
    //是否允许同时支持多个手势，默认是不支持多个手势
    // 返回yes表示支持多个手势
    - (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
    {
        return YES;
    }
    // 是否允许接收手指的触摸点
    - (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch
    {
    //    NSLog(@"%@",touch);
        return YES;
    }
```
## 逐个实现每个手势

### 1、点按 UITapGestureRecognizer

```objc
    #pragma mark - 各种手势
    // UITapGestureRecognizer
    // 点按
    - (void)tapImage
    {
        // 创建点按手势
        UITapGestureRecognizer *gestur = [[UITapGestureRecognizeralloc] initWithTarget:selfaction:@selector(tap:)];
        // 给imageView添加手势，默认imageView不响应事件
        gestur.delegate = self;
        gestur.numberOfTapsRequired = 2; // 点击次数
        [self.imageView addGestureRecognizer:gestur];
    }
    // 响应点按手势事件
    - (void)tap:(UITapGestureRecognizer *)tap
    {
        // 连续点按两次就更换显示图片
        if (tap.numberOfTapsRequired == 2) {

            self.imageView.image = [UIImage imageNamed:@"qianbao"];
        }
    }
```

### 2、捏合 可以使用两个手指进行缩放 UIPinchGestureRecognizer

 ```objc
    //UIPinchGestureRecognizer
    // 捏合，可以缩放
    - (void)pinchImage
    {
        UIPinchGestureRecognizer *pinch = [[UIPinchGestureRecognizeralloc] initWithTarget:selfaction:@selector(pinch:)];
       //pinch.delegate = self;

        [self.imageViewaddGestureRecognizer:pinch];
    }
    - (void)pinch:(UIPinchGestureRecognizer *)pinch
    {
        self.imageView.transform = CGAffineTransformScale(self.imageView.transform, pinch.scale,pinch.scale);
        // 复位
        pinch.scale = 1;
    }
```

### 3、长按 UILongPressGestureRecognizer

```objc
    // UILongPressGestureRecognizer
    // 长按,默认会触发两次,开始时调用一次，结束时调用一次，可以根据手势状态
    - (void)longPressImage
    {
        UILongPressGestureRecognizer *longPress = [[UILongPressGestureRecognizeralloc] initWithTarget:selfaction:@selector(longPress:)];
        [self.imageView addGestureRecognizer:longPress];
    }

    - (void)longPress:(UILongPressGestureRecognizer *)longP
    {
        // 判断手势状态，只在手势开始的时候执行。
        if(longP.state == UIGestureRecognizerStateBegan)
        {
            self.imageView.transform = CGAffineTransformRotate(self.imageView.transform, M_PI);
        }
    }
```

### 4、清扫 轻轻向某个方向滑动

```objc
    // UISwipeGestureRecognizer
    // 清扫手势，默认向右
    - (void)swipeImage
    {
        UISwipeGestureRecognizer *swipe = [[UISwipeGestureRecognizeralloc] initWithTarget:selfaction:@selector(swipe:)];
        [self.imageViewaddGestureRecognizer:swipe];

        // 如果以后想要一个控件支持多个方向的轻扫，必须创建多个轻扫手势，一个轻扫手势只支持一个方向
        // 默认轻扫的方向是往右
        UISwipeGestureRecognizer *swipeUp = [[UISwipeGestureRecognizeralloc] initWithTarget:selfaction:@selector(swipe:)];
        swipeUp.direction = UISwipeGestureRecognizerDirectionUp;
        [self.imageView addGestureRecognizer:swipeUp];
    }
    - (void)swipe:(UISwipeGestureRecognizer *)swipe
    {
        // 输出状态
        NSLog(@"%s\n%ld",__func__,swipe.direction);
```

### 5、旋转 UIRotationGestureRecognizer

```objc
    // UIRotationGestureRecognizer
    // 旋转手势
    - (void)rotateImage
    {
        UIRotationGestureRecognizer *rotate = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotate:)];
        [self.imageView addGestureRecognizer:rotate];
    }
    - (void)rotate:(UIRotationGestureRecognizer *)rotate
    {
        self.imageView.transform = CGAffineTransformRotate(self.imageView.transform, rotate.rotation);
        // 复位
        rotate.rotation = 0;
    }
```

### 6、拖拽  UIPanGestureRecognizer

```objc
    // 拖拽
    - (void)panImage
    {
        UIPanGestureRecognizer *pan = [[UIPanGestureRecognizeralloc] initWithTarget:selfaction:@selector(pan:)];

        [self.imageViewaddGestureRecognizer:pan];
    }
    - (void)pan:(UIPanGestureRecognizer *)pan
    {
        // 获取手势移动点
        CGPoint curP = [pan translationInView:self.imageView];

        self.imageView.transform = CGAffineTransformTranslate(self.imageView.transform, curP.x, curP.y);
        // 复位,一定要复位
        [pan setTranslation:CGPointZeroinView:self.imageView];
    }
```
- 在视图加载时进行调用即可

```objc
    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view, typically from a nib.
        // 设置imageView 响应事件
        self.imageView.userInteractionEnabled = YES;
        // 点按图形
        [self tapImage];
        // 长按
        [selflongPressImage];
        // 清扫
        [self swipeImage];
        // 旋转
        [self rotateImage];
        // 缩放
        [self pinchImage];
        // 拖拽
    //    [self panImage];
    }
```

## 手势使用例子

### 抽屉效果的实现

- 1、需要三个视图，一个主视图，两个底部视图

```objc
    @property (nonatomic,weak) UIView *mainView;
    @property (nonatomic,weak) UIView *leftView;
    @property (nonatomic,weak) UIView *rightView;
```

- 2、初始化界面

```objc
    // 初始化界面
    - (void)initUI
    {
        // 左划显示视图
        UIView *left = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
        self.leftView = left;
        self.leftView.backgroundColor  = [UIColorblueColor];
        [self.view addSubview:self.leftView];
        // 右划显示视图
        UIView *right = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
        self.rightView = right;
        self.rightView.backgroundColor  = [UIColorgreenColor];
        [self.view addSubview:self.rightView];
        // 主视图
        UIView *main = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
        self.mainView = main;
        self.mainView.backgroundColor  = [UIColorredColor];
        [self.view addSubview:self.mainView];
    }
```

-  3、添加手势

```objc
    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view, typically from a nib.

        // 初始化界面
        [self initUI];
        //添加手势
        UIPanGestureRecognizer *pan = [[UIPanGestureRecognizeralloc] initWithTarget:selfaction:@selector(pan:)];
        [self.view addGestureRecognizer:pan];
    }
```
- 手势监听方法

```objc
    #define kLeft -250
    #define kRight 275
    - (void)pan:(UIPanGestureRecognizer *)pan
    {
        //获取当前点
        CGPoint curP = [pan translationInView:self.view]; // 转换指定view的位置
        // x轴偏移，y轴改变，宽度高度都跟着改变
        self.mainView.frame = [self frameWithOffsetX:curP.x];
        // 更新几个视图的状态
        [self observeValueForKeyPath:nil ofObject:nil change:nil context:nil];
        // 复位
        [pan setTranslation:CGPointZero inView:self.view];

        // 定位,手势结束后进行判断
        if (pan.state == UIGestureRecognizerStateEnded) {
            // 左划
            CGFloat target = 0;
            //  如果向右滑动操作屏幕的一半就定位到 275
            if (self.mainView.frame.origin.x > kWidth * 0.5) {
                target = kRight;
            }
            // 否则就定位到 -250
            else if(CGRectGetMaxX(self.mainView.frame) < kWidth * 0.5)
            {
                target = kLeft;
            }

            // 获得x轴的偏移量
            CGFloat offsetX = target - self.mainView.frame.origin.x ;

           [UIView animateWithDuration:0.25 animations:^{

               if (target == 0)
               {
                   self.mainView.frame = self.view.bounds;
               }
               else
               {
                   self.mainView.frame = [self frameWithOffsetX:offsetX];
               }
           }];
        }
    }
```
- 每次滑动时都要判断要显示的底部视图，显示left还是right,根据x值就可以判断

```objc
    // 监听frame属性的改变
    // 只要监听的属性一改变，就会调用观察者的这个方法，通知你有新值
    - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
    {
        // 更新几个视图的状态
        // 判断x坐标大小，大于0向右滑动，小于0 向左滑动
        if(self.mainView.frame.origin.x > 0)
        {
            self.rightView.hidden = YES;
            self.leftView.hidden = NO;
        }
        else
        {
            self.rightView.hidden = NO;
            self.leftView.hidden = YES;
        }
    }
```

- 每次滑动都要随时判断frame的状态，在方法 frameWithOffsetX 中进行判断

```objc
    #define kWidth 80 // 距离顶部最大距离
    // 计算frame的位置信息
    - (CGRect)frameWithOffsetX:(CGFloat)offsetX
    {
        CGRect frame = self.mainView.frame;
        // 获得x轴偏移量
        frame.origin.x += offsetX;
        // 获得屏幕宽度好高度
        CGFloat screenW = [UIScreenmainScreen].bounds.size.width;
        CGFloat screenH = [UIScreenmainScreen].bounds.size.height;

        // y轴偏移量
        CGFloat offsetY = offsetX * kWidth / screenW;

        // 获得上一次的高度
        CGFloat preH = frame.size.height;
        // 获得上一次的宽度
        CGFloat preW = frame.size.width;

        // 当前的高度
        CGFloat curH = preH - 2 * offsetY;
        if (frame.origin.x < 0) {
            curH = preH + 2 * offsetY;
        }
        // 获取缩放比例
        CGFloat scale = curH / preH;
        // 当前的宽度
        CGFloat curW = preW * scale;
        // 设置frame
        frame.origin.y = (screenH - curH) / 2;
        frame.size.height = curH;
        frame.size.width = curW;

        return frame;
    }
```

- 4、自动提示宏

 - 在使用KVO对某个属性进行监听时，如果参数直接传入字符串，很有可能写错。可以使用自动提示宏来自动提示要监视的属性。

```objc
    // 使用KVO时刻监听属性的改变
    // Observer:观察者 谁想监听
    // KeyPath：监听的属性
    // options:监听新值的改变
    [self.mainViewaddObserver:selfforKeyPath:keyPath(self.mainView, frame) options:NSKeyValueObservingOptionNewcontext:nil];
    // 为了防止给属性添加监视器时写错，可以使用自动提示宏来传参数
    [self.mainView addObserver:self forKeyPath:@“frame”options:NSKeyValueObservingOptionNew context:nil];
在方法  observeValueForKeyPath  中对属性改变进行监听。
```

## 宏的书写规则

```objc
// 自动提示宏
// 宏的操作原理，每输入一个字母就会直接把宏右边的拷贝，并且会自动补齐前面的内容。
// 宏里面的#，会自动把后面的参数变成C语言的字符串,
// (obj.keyPath,keyPath) 逗号表达式，最终结果取keyPath
// void表示不使用这个参数的返回结果
// @() 表示把c语言字符串转换成OC字符串
#define keyPath(obj,keyPath) @(((void)obj.keyPath,#keyPath))
```


