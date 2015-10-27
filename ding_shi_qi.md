# 定时器
## CADisplayLink 和 NSTiemr

- CADisplayLink
    - 适用于UI动画的重绘。精度比较高
    - iOS设备的屏幕刷新频率是60Hz，也就是每秒60次。这个定时器的执行频率和屏幕刷新频率一致。但是可以通过修改`frameInterval`来改变定时器的调用频率。
- NTimer 
    - 精度比较低，如果当前线程阻塞，NSTimer不会起作用
    - NSTimer初始化器接受调用方法逻辑之间的间隔作为它的其中一个参数，预设一秒执行30次。CADisplayLink默认每秒运行60次，通过它的frameInterval属性改变每秒运行帧数，如设置为2，意味CADisplayLink每隔一帧运行一次，有效的逻辑每秒运行30次。
    - 此外，NSTimer接受另一个参数是否重复，而把CADisplayLink设置为重复（默认重复？）直到它失效。
    - 还有一个区别在于，NSTimer一旦初始化它就开始运行，而CADisplayLink需要将显示链接添加到一个运行循环中，即用于处理系统事件的一个Cocoa Touch结构。
    - NSTimer 我们通常会用在背景计算，更新一些数值资料，而如果牵涉到画面的更新，动画过程的演变，我们通常会用CADisplayLink。
    - 但是要使用CADisplayLink，需要加入QuartzCore.framework及#import <QuartzCore/CADisplayLink.h>

## 源文件分析

```objc
/* CoreAnimation - CADisplayLink.h

   Copyright (c) 2009-2015, Apple Inc.
   All rights reserved. */

#import <QuartzCore/CABase.h>
#import <Foundation/NSObject.h>

@class NSString, NSRunLoop;

/** Class representing a timer bound to the display vsync. **/

@interface CADisplayLink : NSObject
{
@private
  void *_impl;
}

/* Create a new display link object for the main display. It will
 * invoke the method called 'sel' on 'target', the method has the
 * signature '(void)selector:(CADisplayLink *)sender'. */

+ (CADisplayLink *)displayLinkWithTarget:(id)target selector:(SEL)sel;

/* Adds the receiver to the given run-loop and mode. Unless paused, it
 * will fire every vsync until removed. Each object may only be added
 * to a single run-loop, but it may be added in multiple modes at once.
 * While added to a run-loop it will implicitly be retained. */

- (void)addToRunLoop:(NSRunLoop *)runloop forMode:(NSString *)mode;

/* Removes the receiver from the given mode of the runloop. This will
 * implicitly release it when removed from the last mode it has been
 * registered for. */

- (void)removeFromRunLoop:(NSRunLoop *)runloop forMode:(NSString *)mode;

/* Removes the object from all runloop modes (releasing the receiver if
 * it has been implicitly retained) and releases the 'target' object. */

- (void)invalidate;

/* The current time, and duration of the display frame associated with
 * the most recent target invocation. Time is represented using the
 * normal Core Animation conventions, i.e. Mach host time converted to
 * seconds. */
// 但是 duration只是个大概的时间，如果CPU忙于其它计算，就没法保证以相同的频率执行屏幕的绘制操作，这样会跳过几次调用回调方法的机会。 frameInterval属性是可读可写的NSInteger型值，标识间隔多少帧调用一次selector 方法，默认值是1，即每帧都调用一次。
@property(readonly, nonatomic) CFTimeInterval timestamp;
@property(readonly, nonatomic) CFTimeInterval duration;

/* When true the object is prevented from firing. Initial state is
 * false. */

@property(getter=isPaused, nonatomic) BOOL paused;

/* Defines how many display frames must pass between each time the
 * display link fires. Default value is one, which means the display
 * link will fire for every display frame. Setting the interval to two
 * will cause the display link to fire every other display frame, and
 * so on. The behavior when using values less than one is undefined. */

@property(nonatomic) NSInteger frameInterval;

@end
```
- 真正可用的几个属性和方法是
    - 创建对象 `+ (CADisplayLink *)displayLinkWithTarget:(id)target selector:(SEL)sel;`
    - 添加到主运行循环 `- (void)addToRunLoop:(NSRunLoop *)runloop forMode:(NSString *)mode;`
    - 从主运行循环移除 `- (void)removeFromRunLoop:(NSRunLoop *)runloop forMode:(NSString *)mode;`
    - 释放定时器 `- (void)invalidate;`
    - 定时器状态,默认是false  `@property(getter=isPaused, nonatomic) BOOL paused;`
    - 刷新频率,设置这个参数来控制定时器刷新频率 `@property(nonatomic) NSInteger frameInterval;`

- 使用方式

```
- (void)viewDidLoad {
    [super viewDidLoad];

    // 初始化
    self.displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(updateColor)];
    // 添加到Runloop
    [self.displayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
    self.displayLink.frameInterval = 2; // 设置刷新频率，默认是每秒60次，设置为2表示每秒30次

}
- (void)updateColor
{
    NSArray *colors = @[[UIColor redColor],[UIColor blueColor],[UIColor greenColor]];
    self.view.backgroundColor = colors[arc4random_uniform(3)];
    NSLog(@"%zd--%zd",self.displayLink.timestamp,self.displayLink.duration);
}
// 手动关闭定时器
- (void)dealloc
{
    self.displayLink.paused = true;
    [self.displayLink invalidate];
    self.displayLink = nil;
}

```

## NSTimer

- 源代码分析

```
/*	NSTimer.h
	Copyright (c) 1994-2014, Apple Inc. All rights reserved.
*/

#import <Foundation/NSObject.h>
#import <Foundation/NSDate.h>

@interface NSTimer : NSObject

+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;

+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(id)userInfo repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(id)userInfo repeats:(BOOL)yesOrNo;

- (instancetype)initWithFireDate:(NSDate *)date interval:(NSTimeInterval)ti target:(id)t selector:(SEL)s userInfo:(id)ui repeats:(BOOL)rep NS_DESIGNATED_INITIALIZER;

- (void)fire;

@property (copy) NSDate *fireDate;
@property (readonly) NSTimeInterval timeInterval;

// Setting a tolerance for a timer allows it to fire later than the scheduled fire date, improving the ability of the system to optimize for increased power savings and responsiveness. The timer may fire at any time between its scheduled fire date and the scheduled fire date plus the tolerance. The timer will not fire before the scheduled fire date. For repeating timers, the next fire date is calculated from the original fire date regardless of tolerance applied at individual fire times, to avoid drift. The default value is zero, which means no additional tolerance is applied. The system reserves the right to apply a small amount of tolerance to certain timers regardless of the value of this property.
// As the user of the timer, you will have the best idea of what an appropriate tolerance for a timer may be. A general rule of thumb, though, is to set the tolerance to at least 10% of the interval, for a repeating timer. Even a small amount of tolerance will have a significant positive impact on the power usage of your application. The system may put a maximum value of the tolerance.
@property NSTimeInterval tolerance NS_AVAILABLE(10_9, 7_0);

- (void)invalidate;
@property (readonly, getter=isValid) BOOL valid;

@property (readonly, retain) id userInfo;

@end
```

- 方法和属性
    - 构造方法 
    - `+scheduledTimerWithTimeInterval::`
    - `+timerWithTimeInterval::`
    - `-initWithFireDate:`
    - 启动 `- (void)fire;`
    - 设置启动时间 `fireDate;`
    - 间隔时间 `timeInterval`
    - 最大可容忍时间 `tolerance` 在容忍时间内调用定时器
    - 释放定时器，主要是从runloop移除 `invalidate`
    - 传递用户数据 `userInfo`
- 使用方法

```
- (void)viewDidLoad {
    [super viewDidLoad];
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(updateColor) userInfo:nil repeats:YES];
    
    [[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSDefaultRunLoopMode];

}
- (void)updateColor
{
    NSArray *colors = @[[UIColor redColor],[UIColor blueColor],[UIColor greenColor]];
    self.view.backgroundColor = colors[arc4random_uniform(3)];
    NSLog(@"%zd--%zd",self.displayLink.timestamp,self.displayLink.duration);
}
```