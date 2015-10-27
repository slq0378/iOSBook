# 定时器
- CADisplayLink
- 适用于UI动画的重绘。精度比较高，和屏幕刷新频率一致
- NTimer 
- 精度比较低，如果当前线程阻塞，NSTimer不会起作用
    
- NSTimer初始化器接受调用方法逻辑之间的间隔作为它的其中一个参数，预设一秒执行30次。CADisplayLink默认每秒运行60次，通过它的frameInterval属性改变每秒运行帧数，如设置为2，意味CADisplayLink每隔一帧运行一次，有效的逻辑每秒运行30次。

- 此外，NSTimer接受另一个参数是否重复，而把CADisplayLink设置为重复（默认重复？）直到它失效。

- 还有一个区别在于，NSTimer一旦初始化它就开始运行，而CADisplayLink需要将显示链接添加到一个运行循环中，即用于处理系统事件的一个Cocoa Touch结构。

- NSTimer 我们通常会用在背景计算，更新一些数值资料，而如果牵涉到画面的更新，动画过程的演变，我们通常会用CADisplayLink。

- 但是要使用CADisplayLink，需要加入QuartzCore.framework及#import <QuartzCore/CADisplayLink.h>


