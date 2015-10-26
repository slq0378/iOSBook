# 事件处理

## IOS事件处理
- 1、触摸事件
- 2、加速器事件：重力感应，旋转等事件
- 3、远程遥控事件：蓝牙线控，耳机线控、通知等

## 触摸事件
- 响应者对象
- 只有继承了UIResponder得对象才能接收并处理事件
- 常见类有：UIApplication、UIViewController、UIView

## UIResponder中几个事件响应方法

- 触摸事件方法

```objc
    - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
    - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
    - (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
    - (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```

- 加速器事件方法

```objc
    - (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event NS_AVAILABLE_IOS(3_0);
    - (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event NS_AVAILABLE_IOS(3_0);
    - (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event NS_AVAILABLE_IOS(3_0);
```

- 遥控事件方法

```objc
    - (void)remoteControlReceivedWithEvent:(UIEvent *)event NS_AVAILABLE_IOS(4_0);
```
- 这些方法在响应事件时需要自己重写方法。


- UIView中的触摸事件
    - 自定义view，实现几个方法，如果要响应UIView的触摸事件，需要自定控件的CustomClass为自定义的类，然后再自定义的类中实现以下方法。

```objc
    // 触摸开始
    - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
    {
        NSLog(@"%s",__func__);
        NSLog(@"%@",touches);
        NSLog(@"----%@",event);
    }
    // 移动手指
    - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
    {
        NSLog(@"%s",__func__);
        // 获取UITouch
        UITouch *touch = [touches anyObject];
        // 获取偏移量
        CGPoint location = [touch locationInView:self];
        CGPoint previous = [touch previousLocationInView:self];
        CGFloat offsetX = location.x - previous.x;
        CGFloat offsetY = location.y - previous.y;
        // 移动view
        self.transform = CGAffineTransformTranslate(self.transform, offsetX, offsetY);
    }
    // 触摸结束
    - (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event
    {
        NSLog(@"%s",__func__);
    }
    // 系统事件中断：短信，电话等
    - (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event
    {
        NSLog(@"%s",__func__);
    }
```

## UITouch 触摸对象
- 触摸对象
    - 当用户用一根触摸屏幕时，会创建一个与手指相关联的UITouch对象
    - 一根手指对应一个UITouch对象UITouch的作用保存着跟手指相关的信息，比如触摸的位置、时间、阶段
    - 当手指移动时，系统会更新同一个UITouch对象，使之能够一直保存该手指在的触摸位置
    - 当手指离开屏幕时，系统会销毁相应的UITouch对象
    - 提示：iPhone开发中，要避免使用双击事件！

- **几个常用属性**

```objc
    // 触摸产生时所处的窗口
    @property(nonatomic,readonly,retain) UIWindow    *window;
    // 触摸产生时所处的视图
    @property(nonatomic,readonly,retain) UIView      *view;
    // 短时间内点按屏幕的次数，可以根据tapCount判断单击、双击或更多的点击
    @property(nonatomic,readonly) NSUInteger          tapCount;
    // 记录了触摸事件产生或变化时的时间，单位是秒
    @property(nonatomic,readonly) NSTimeInterval      timestamp;
    // 当前触摸事件所处的状态
    @property(nonatomic,readonly) UITouchPhase        phase;
```

- **常用的几个方法**

```objc
    /*返回值表示触摸在view上的位置
     这里返回的位置是针对view的坐标系的（以view的左上角为原点(0, 0)）
     调用时传入的view参数为nil的话，返回的是触摸点在UIWindow的位置*/
    - (CGPoint)locationInView:(UIView *)view;
    //该方法记录了前一个触摸点的位置
    - (CGPoint)previousLocationInView:(UIView *)view;
```

## UIEvent 事件对象

```objc
    //每产生一个事件，就会产生一个UIEvent对象
    //UIEvent：称为事件对象，记录事件产生的时刻和类型
    //常见属性
    //事件类型
    @property(nonatomic,readonly) UIEventType     type;
    @property(nonatomic,readonly) UIEventSubtype  subtype;
    //事件产生的时间
    @property(nonatomic,readonly) NSTimeInterval  timestamp;
    //UIEvent还提供了相应的方法可以获得在某个view上面的触摸对象（UITouch）
```

- 每个触摸事件都会传入一个UITouch对象和一个UIEvent对象，其内部组成如下，可以通过这两个参数控制控件的运动。
屏幕快照 2015 06 16 13 09 32


    一次完整的触摸过程中，只会产生一个事件对象，4个触摸方法都是同一个event参数
    如果两根手指同时触摸一个view，那么view只会调用一次touchesBegan:withEvent:方法，touches参数中装着2个UITouch对象
    如果这两根手指一前一后分开触摸同一个view，那么view会分别调用2次touchesBegan:withEvent:方法，并且每次调用时的touches参数中只包含一个UITouch对象
    根据touches中UITouch的个数可以判断出是单点触摸还是多点触摸

- 事件的产生和传递
    - 1、发生触摸事件后，系统会将该事件加入到一个由UIApplication管理的事件队列中
    - 2、UIApplication会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口（keyWindow）
    - 3、主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件，这也是整个事件处理过程的第一步

- **触摸事件的传递顺序是从父控件到子控件**

- 如何找到最合适的控件来处理事件？
    - 1、自己是否能接收触摸事件？
    - 2、触摸点是否在自己身上？
    - 3、从后往前遍历子控件，重复前面的两个步骤
    - 4、如果没有符合条件的子控件，那么就自己最适合处

- UIView 不接收事件的三种情况
    - 1、不接收用户交互
        - userInteractionEnabled = NO;
    - 2、隐藏
        - hidden = YES;
    - 3、透明
        - alpha = 0.0 ~ 0.01

- 这三种情况下，不响应事件，UIImageView默认userInteractionEnabled位NO。

- 事件传递的完整过程
    - 1、先将事件对象由上往下传递(由父控件传递给子控件)，找到最合适的控件来处理这个事件。
    - 2、调用最合适控件的touches….方法
    - 3、如果调用了[super touches….];就会将事件顺着响应者链条往上传递，传递给上一个响应者
    - 4、接着就会调用上一个响应者的touches….方法

- hitTest和pointInside的使用

```objc
    // 事件传递的时候调用
    // 什么时候调用:当事件传递给控件的时候，就会调用控件的这个方法，去寻找最合适的view
    // 作用：寻找最合适的view
     //point：当前的触摸点，point这个点的坐标系就是方法调用者
    - (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
    {
        // 调用系统的做法去寻找最合适的view，返回最合适的view
        UIView *fitView = [super hitTest:point withEvent:event];
        NSLog(@"fitView--%@",fitView);
        return fitView;
    }
    // 作用：判断当前这个点在不在方法调用者（控件）上
    - (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
    {
        return YES;
    }
```

- 自定义hitTest文件的实现

```objc
    /**
     *  自定义hitTest方法
     */
    - (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
    {
    //    UIView *fview =  [super hitTest:point withEvent:event];
    //
    //    NSLog(@"fview----%@",[self class]);
    //    return fview;
        // 自己实现
        // 1、检查view是否能接收事件
        if(!self.userInteractionEnabled || self.hidden  || self.alpha <= 0.01)
        {
            return nil;
        }
        // 2、检查触摸点是否在自己身上
        if (![self pointInside:point withEvent:event])
        {
            return nil;
        }
        // 3、从后往前遍历所有子控件是否满足以上两条
        NSInteger count = self.subviews.count;
        for (NSInteger i = count; i > 0; i --)
        {
            UIView *childView = self.subviews[i];
            // 将父控件上得点转换成子控件身上的点
            CGPoint childPoint = [self convertPoint:point toView:childView];
            // 递归子控件
            UIView *fitView = [childView hitTest:childPoint withEvent:event];
            if (fitView)
            {
                return childView;
            }
        }
        NSLog(@"fview----%@",[self class]);
        // 如果找不到比自己更合适的就返回自己
        return self;
    }
```
- 由此可见寻找最合适的view的过程就是递归遍历所有控件，找到最合适的view。

- 多层视图时如何只响应其中一个

```objc
    // 多层视图时如何只响应其中一个
    - (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
    {
    //    UIView *fview =  [super hitTest:point withEvent:event];
    //
    //    NSLog(@"fview----%@",[self class]);
    //    return fview;
        // 1、判断点是否在自己身上
        CGPoint selfPoint = [self convertPoint:point toView:self.btn];
        // 判断点是否也在按钮上
        if([self.btn pointInside:selfPoint withEvent:event])// 如果点也在按钮身上身上，就相应按钮
        {
            NSLog(@"btn---%@",self);
            return self.btn;
        }
        else// 否则就继续判断父控件
        {
            return [super hitTest:point withEvent:event];
        }
    }
```

