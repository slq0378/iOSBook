# RunLoop

## 基本作用
- 保持程序运行状态。简单一点就是在main函数里加入一个死循环，使得程序不会退出。
- 处理各种app中的事件（比如触摸、定时器事件、selector事件等）。
- 节省CPU资源，有事做就做事，没事做就休息。

```objc
int main(int argc, char * argv[]) {
    @autoreleasepool {
        // 在这里创建一个RunLoop循环
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
## 使用
- `Foundation` ->` NSRunLoop`
- `Core Foundation` -> `CFRunLoopRef`
- 其中`NSRunLoop`是对`CFRunLoopRef`的封装，而且`CFRunLoopRef`是开源的，可以研究这个类

## RunLoop与线程
- 每条线程有一个唯一的RunLoop对象。
- 主线程的RunLoop对象已经有系统创建好，子线程的需要手动创建。
- 而子线程的RunLoop对象在第一次获取是创建，在线程结束时销毁。

```objc
	// Foundation
    [NSRunLoop currentRunLoop]; // 当前RunLoop,在子线程中激活RunLoop
    [NSRunLoop mainRunLoop]; // 主线程的RunLoop
    // Core Foundation
    CFRunLoopGetCurrent(); // 当前RunLoop,
    CFRunLoopGetMain(); // 主线程的RunLoop

```
## RunLoop相关类

- Core Foundation中关于RunLoop的5个类
	- CFRunLoopRef - 一个RunLoop包含多个Mode
	- CFRunLoopModeRef - 运行模式,包含多个source、timer、observe
	- CFRunLoopSourceRef - 事件源（触摸、点击、拖动等）
	- CFRunLoopTimerRef - 定时器事件
	- CFRunLoopObserverRef - 观察者，监视runloop状态

- 类之间的关系
	- 每次启动程序都会指定一个Mode，如果要切换Mode，只退出Loop，再进入新mode

![](/images/RunLoop.png)

### CFRunLoopModeRef
- 系统默认注册了5个mode
	- **kCFRunLoopDefaultMode**：App的默认Mode，通常主线程是在这个Mode下运行
	- **UITrackingRunLoopMode**：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
	- UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
	- GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
	- **kCFRunLoopCommonModes**: 这是一个占位用的Mode，不是一种真正的Mode
- 其中 前两个可以使用，后面几个都是系统使用的。最后一个可以在夸mode使用定时器时指定。
- 在控制器中添加一个scrollView对象，下面添加的定时器会跟模式有关系，

```objc
  NSTimer *timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(run) userInfo:nil repeats:YES];
    [[NSRunLoop mainRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode]; // 默认模式
    [[NSRunLoop mainRunLoop] addTimer:timer forMode:NSRunLoopCommonModes]; // 滚动模式也相应事件
```

 - NSRunLoopCommonModes - 默认支持系统支持的模式，包括两种kCFRunLoopDefaultMode，UITrackingRunLoopMode，也就是说如果想在滚动模式下监听事件，就要设置成这个参数。可以在控制台查看系统支持的模式。


### CFRunLoopSourceRef
- 事件源 (输入源)
- Source0:非基于Port
- Source1:基于Port，通过内核和其他线程通信，接收事件

### CFRunLoopTimerRef
- 时间触发器
	- 各种定时器NSTimer

### CFRunLoopObserverRef
- 观察者：监听RunLoop的状态
- 可用状态

```objc
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0), // 1,即将进入RunLoop
    kCFRunLoopBeforeTimers = (1UL << 1), // 2，处理定时器之前
    kCFRunLoopBeforeSources = (1UL << 2), // 4，处理事件之前
    kCFRunLoopBeforeWaiting = (1UL << 5), // 32，睡眠之前
    kCFRunLoopAfterWaiting = (1UL << 6), // 64，唤醒之后
    kCFRunLoopExit = (1UL << 7), // 128，退出RunLoop
    kCFRunLoopAllActivities = 0x0FFFFFFFU // 所有状态
};

```
- 自定义observer
	- 要手动创建观察者对象，并将观察者添加在RunLoop中去。

```objc
CFRunLoopObserverRef CFRunLoopObserverCreateWithHandler (
CFAllocatorRef allocator, //分配器
CFOptionFlags activities, // 要监听状态
Boolean repeats, // 重复与否
CFIndex order, // 索引
void (^block)( CFRunLoopObserverRef observer,
CFRunLoopActivity activity)// block ,状态触发后进入这个block
);

    // 1 创建观察者对象 observer
   CFRunLoopObserverRef observer =  CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(), kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
        NSLog(@"---observer----%zd",activity);
    });
    // 2 添加监听者
    CFRunLoopAddObserver(CFRunLoopGetMain(), observer, kCFRunLoopDefaultMode);
    // 3 使用过要手动释放
    CFRelease(observer);
```
- **注意事项**
    - CF的内存管理（Core Foundation）
    - 凡是带有Create、Copy、Retain等字眼的函数，创建出来的对象，都需要在最后做一次release
    - 比如CFRunLoopObserverCreate
    - release函数：CFRelease(对象);

## RunLoop逻辑
- RunLoop逻辑官方版

![](/images/RunLoop官方版.png)
- RunLoop逻辑

![](/images/RunLoop处理逻辑.png)

## 常驻线程
- 如果想在程序启动后保持一个线程处于后台处理各种事件，那么可以使用常驻线程。
- 可以指定模式为NSRunLoopCommonModes，然后就可以在默认和滚动下监听事件了。

```objc
    // 添加到RunLoop，保持线程不销毁,添加一个事件源（source或者timer）即可保持不死
    [[NSRunLoop currentRunLoop] addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
    [[NSRunLoop currentRunLoop] run];
```
然后在其他情况下照样处理各种事件

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    [self performSelector:@selector(runInThread) onThread:self.thread withObject:nil waitUntilDone:NO modes:@[NSDefaultRunLoopMode]];
}
- (void)runInThread
{
    NSLog(@"runInThread---%@",[NSThread currentThread]);
}
```
## RunLoop 面试题

- 什么是RunLoop？
	- 从字面意思看：运行循环、跑圈
	- 其实它内部就是do-while循环，在这个循环内部不断地处理各种任务（比如Source、Timer、Observer）
	- 一个线程对应一个RunLoop，主线程的RunLoop默认已经启动，子线程的RunLoop得手动启动（调用run方法）
	- RunLoop只能选择一个Mode启动，如果当前Mode中没有任何Source、Timer、Observer，那么就直接退出RunLoop

- 自动释放池什么时候释放？
	- 通过Observer监听RunLoop的状态，一旦监听到RunLoop即将进入睡眠等待状态，就释放自动释放池（kCFRunLoopBeforeWaiting）

```
kCFRunLoopEntry; // 创建一个自动释放池
kCFRunLoopBeforeWaiting; // 销毁自动释放池，创建一个新的自动释放池
kCFRunLoopExit; // 销毁自动释放池
```

- 在开发中如何使用RunLoop？什么应用场景？
	- 开启一个常驻线程（让一个子线程不进入消亡状态，等待其他线程发来消息，处理其他事件）
		- 在子线程中开启一个定时器
		- 在子线程中进行一些长期监控
	- 可以控制定时器在特定模式下执行
	- 可以让某些事件（行为、任务）在特定模式下执行
	- 可以添加Observer监听RunLoop的状态，比如监听点击事件的处理（在所有点击事件之前做一些事情）

## GCD定时器

- GCD定时器，比NSTimer更加准确,并且生存周期和RunLoop无关


```objc
    // 定时器队列
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
//     dispatch_queue_t queue = dispatch_get_main_queue();
    // 创建定时器
    self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    // 设置定时器属性
    // 参数1：定时器
    // 参数2：起始时间，在当前时间的基础上晚上几秒
    // 参数3：间隔时间
    // 设置定时器的各种属性（几时开始任务，每隔多长时间执行一次）
    // GCD的时间参数，一般是纳秒（1秒 == 10的9次方纳秒）
    // 何时开始执行第一个任务
    // dispatch_time(DISPATCH_TIME_NOW, 2.0 * NSEC_PER_SEC) 比当前时间晚2秒
    dispatch_time_t startTime = dispatch_time(DISPATCH_TIME_NOW, 2 * NSEC_PER_SEC);
    dispatch_source_set_timer(self.timer, startTime, 1 * NSEC_PER_SEC, 0);

    // 设置回调
    dispatch_source_set_event_handler(self.timer, ^{
        NSLog(@"---GCD---%@",[NSThread currentThread]);
        count ++;
        if(count == 10)
        {
             NSLog(@"---dispatch_cancel---%@",[NSThread currentThread]);
            dispatch_cancel(self.timer);
        }
    });
    dispatch_resume(self.timer);
```
