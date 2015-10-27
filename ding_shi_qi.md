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

```
