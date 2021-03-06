
## iOS动画相关类

![](iOS动画相关类.png)

## 图层类
- CALayer 是所有图层类的基础，它使所有核心动画图层类的父类。
- 所有的核心动画的动画类都从`CAAnimation`类继承而来。
	- `CAAnimation` 遵守 `CAMediaTiming`协议，提供动画持续时间，速度，重复计数。
	- `CAAnimation` 遵守 `CAAction` 协议来实现图层动画的响应。
- 具体的`CAAnimation`子类
	- `CATransition` 提供了图层变化的过渡效果，包括淡入淡出`fade`、push、reveal等
	- `CAAnimationGroup` 允许添加一系列动画效果组合起来，并行显示动画。
	- `CAPropertyAnimation` 是支持动画的显示图层的关键路径中指定的属性。
	- `CABasicAnimation` 简单的修改图层属性的动画类
	- `CAKeyframeAnimation` 关键帧动画
- 布局管理类
	- `CAConstraint` 类是一个布局管理器，它可以指定子图层类限制于你指定的约束集合。
- 事务管理类
	- `CATransaction` 是核心动画里面负责协调多个动画原子更新显示操作。事务支持嵌套使用。
	- 事务支持隐式事务和显式事务。

## 核心动画渲染框架
- 图层树
	- 每个可见的图层树由两个相应的树组成:一个是呈现树,一个是渲染树。
	- ![](图层树.png)
	- 图层树就是用户修改的图层属性值，用户一开始修改属性值，会改变这里的值。
	- 呈现树就是在将要显示时才会修改为最新值。
	- 渲染树在渲染图层的时候使用呈现树的值。渲染树负责执行独立于应用活动的复 杂操作。渲染由一个单独的进程或线程来执行,使其对应用程序的运行循环影响最小。
	- 首先一个核心动画开始执行之前，会首先修改图层树中的属性值，然后将要显示是更新呈现树种的属性值，在渲染过程中使用呈现树中的属性值。
	
## 图层的几何和变换
- 坐标系
	- iOS中坐标系是在左上角，向右和向下为正。
	- Mac OS 中是在左下角，向上和向右为正。

### 图层的几何
- 图层的所有几何属性,包括图层的矩阵变换,都可以隐式 和显式动画。
    - ![](图层的几何.png)
    - `position` 位置
	- `bounds` 边界
	- `frame` 位置和尺寸
	- `anchorPoint` 从(0,0)到(1,1)，形变属性会绕着这个点运动

### 几何变换

- `CATransform3D` 的数据结构定义一个同质的三维变换(4x4 CGFloat 值的矩阵),用于 图层的旋转,缩放,偏移,歪斜和应用的透视。
    - 图层属性`transform` (作用于所有图层)和 `sublayerTransform`（只会作用于子图层）
- `CATransform3DIdentity` 是单位矩阵,该矩阵没有缩放、旋转、歪斜、透视。把该 矩阵应用到图层上面,会把图层几何属性修改为默认值。
    - 几个常用方法 
    - `CATransform3D CATransform3DMakeTranslation ( CGFloat tx, CGFloat ty, CGFloat tz );`
    - `CATransform3D CATransform3DMakeScale ( CGFloat sx, CGFloat sy, CGFloat sz );`
    - `CATransform3D CATransform3DMakeRotation ( CGFloat angle, CGFloat x, CGFloat y, CGFloat z );`
    - `CATransform3D CATransform3DScale ( CATransform3D t, CGFloat sx, CGFloat sy, CGFloat sz )`
    - 其中旋转按照弧度计算，不是角度，要自己转换成弧度。
    - `CATransform3DInvert` 一般是用反转 点内转化对象提供反向转换。
- 数据结构

```
/* CoreAnimation - CATransform3D.h

   Copyright (c) 2006-2015, Apple Inc.
   All rights reserved. */

#ifndef CATRANSFORM_H
#define CATRANSFORM_H

#include <QuartzCore/CABase.h>

#ifdef __OBJC__
#import <Foundation/NSValue.h>
#endif

/* Homogeneous three-dimensional transforms. */

struct CATransform3D
{
  CGFloat m11, m12, m13, m14;
  CGFloat m21, m22, m23, m24;
  CGFloat m31, m32, m33, m34;
  CGFloat m41, m42, m43, m44;
};

typedef struct CATransform3D CATransform3D;

CA_EXTERN_C_BEGIN

/* The identity transform: [1 0 0 0; 0 1 0 0; 0 0 1 0; 0 0 0 1]. */

CA_EXTERN const CATransform3D CATransform3DIdentity
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Returns true if 't' is the identity transform. */

CA_EXTERN bool CATransform3DIsIdentity (CATransform3D t)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Returns true if 'a' is exactly equal to 'b'. */

CA_EXTERN bool CATransform3DEqualToTransform (CATransform3D a,
    CATransform3D b)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Returns a transform that translates by '(tx, ty, tz)':
 * t' =  [1 0 0 0; 0 1 0 0; 0 0 1 0; tx ty tz 1]. */

CA_EXTERN CATransform3D CATransform3DMakeTranslation (CGFloat tx,
    CGFloat ty, CGFloat tz)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Returns a transform that scales by `(sx, sy, sz)':
 * t' = [sx 0 0 0; 0 sy 0 0; 0 0 sz 0; 0 0 0 1]. */

CA_EXTERN CATransform3D CATransform3DMakeScale (CGFloat sx, CGFloat sy,
    CGFloat sz)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Returns a transform that rotates by 'angle' radians about the vector
 * '(x, y, z)'. If the vector has length zero the identity transform is
 * returned. */

CA_EXTERN CATransform3D CATransform3DMakeRotation (CGFloat angle, CGFloat x,
    CGFloat y, CGFloat z)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Translate 't' by '(tx, ty, tz)' and return the result:
 * t' = translate(tx, ty, tz) * t. */

CA_EXTERN CATransform3D CATransform3DTranslate (CATransform3D t, CGFloat tx,
    CGFloat ty, CGFloat tz)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Scale 't' by '(sx, sy, sz)' and return the result:
 * t' = scale(sx, sy, sz) * t. */

CA_EXTERN CATransform3D CATransform3DScale (CATransform3D t, CGFloat sx,
    CGFloat sy, CGFloat sz)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Rotate 't' by 'angle' radians about the vector '(x, y, z)' and return
 * the result. If the vector has zero length the behavior is undefined:
 * t' = rotation(angle, x, y, z) * t. */

CA_EXTERN CATransform3D CATransform3DRotate (CATransform3D t, CGFloat angle,
    CGFloat x, CGFloat y, CGFloat z)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Concatenate 'b' to 'a' and return the result: t' = a * b. */

CA_EXTERN CATransform3D CATransform3DConcat (CATransform3D a, CATransform3D b)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Invert 't' and return the result. Returns the original matrix if 't'
 * has no inverse. */

CA_EXTERN CATransform3D CATransform3DInvert (CATransform3D t)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Return a transform with the same effect as affine transform 'm'. */

CA_EXTERN CATransform3D CATransform3DMakeAffineTransform (CGAffineTransform m)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Returns true if 't' can be represented exactly by an affine transform. */

CA_EXTERN bool CATransform3DIsAffine (CATransform3D t)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

/* Returns the affine transform represented by 't'. If 't' can not be
 * represented exactly by an affine transform the returned value is
 * undefined. */

CA_EXTERN CGAffineTransform CATransform3DGetAffineTransform (CATransform3D t)
    __OSX_AVAILABLE_STARTING (__MAC_10_5, __IPHONE_2_0);

CA_EXTERN_C_END

/** NSValue support. **/

#ifdef __OBJC__

NS_ASSUME_NONNULL_BEGIN

@interface NSValue (CATransform3DAdditions)

+ (NSValue *)valueWithCATransform3D:(CATransform3D)t;

@property(readonly) CATransform3D CATransform3DValue;

@end

NS_ASSUME_NONNULL_END

#endif /* __OBJC__ */

#endif /* CATRANSFORM_H */

```

- 通过KVC修改动画属性
    - 核心动画扩展了键-值编码协议,允许通过关键路径获取和设置一个图层的 CATransform3D 矩阵的值。
    - `[self.layer setValue:[NSNumber numberWithInt:1] forKeyPath:@"transform.rotation.x"];`
    - 可以通过`setvalue: forkeypath` 或者 `valueforkeypath` 来设置动画的属性。

## 图层树
- 图层拥有类似UIView的层次结构，拥有父视图（superview）和子视图（subview），`CALayer`拥有父图层（superlayer）和子图层（sublayer）。
- 图层的显示必须依赖window窗口的显示，也就是必须把一个图层托管给一个`UIView`。当视图和图层一起的时候,视图为图层提供了底层的事件处理,而图层为视图 提供了显示的内容。
- iOS 上面的视图系统直接建立在核心动画的图层上面。每个 UIView 的实例会自 动的创建一个 `CALayer` 类的实例,然后把该实例赋值给视图的 `layer` 属性。你可以在 需要的时候向视图的图层里面添加子图层。
- 初始化一个图层并添加到图层树
    - `addSublayer:`
    - `insertSublayer:atIndex:`
    - `￼removeFromSuperlayer`
    - `replaceSublayer:with:`
- 基本属性
    - 这些属性的修改基本上都会产生动画。 
    
```
/** Geometry and layer hierarchy properties. **/

/* The bounds of the layer. Defaults to CGRectZero. Animatable. */

@property CGRect bounds;

/* The position in the superlayer that the anchor point of the layer's
 * bounds rect is aligned to. Defaults to the zero point. Animatable. */

@property CGPoint position;

/* The Z component of the layer's position in its superlayer. Defaults
 * to zero. Animatable. */

@property CGFloat zPosition;

/* Defines the anchor point of the layer's bounds rect, as a point in
 * normalized layer coordinates - '(0, 0)' is the bottom left corner of
 * the bounds rect, '(1, 1)' is the top right corner. Defaults to
 * '(0.5, 0.5)', i.e. the center of the bounds rect. Animatable. */

@property CGPoint anchorPoint;

/* The Z component of the layer's anchor point (i.e. reference point for
 * position and transform). Defaults to zero. Animatable. */

@property CGFloat anchorPointZ;

/* A transform applied to the layer relative to the anchor point of its
 * bounds rect. Defaults to the identity transform. Animatable. */

@property CATransform3D transform;
```
- 自动调整图层大小
