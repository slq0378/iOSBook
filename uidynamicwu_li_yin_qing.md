# UIDynamic 物理引擎

- 任何遵守了UIDynamicItem协议的对象
    - UIView默认已经遵守了UIDynamicItem协议，因此任何UI控件都能做物理仿真
    - UICollectionViewLayoutAttributes类默认也遵守UIDynamicItem协议

- UIDynamic提供了以下几种物理仿真行为
    - UIGravityBehavior：重力行为
    - UICollisionBehavior：碰撞行为
    - UISnapBehavior：捕捉行为
    - UIPushBehavior：推动行为
    - UIAttachmentBehavior：附着行为
    - UIDynamicItemBehavior：动力元素行为


```objc
-(UIDynamicAnimator *)anim
{
    if (!_anim) {
            // 创建物理仿真器(ReferenceView, 参照视图, 其实就是设置仿真范围)
        _anim = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];;
    }
    return  _anim;
}
```

## 重力

- UIGravityBehavior的初始化
    - // item参数 ：里面存放着物理仿真元素
    `- (instancetype)initWithItems:(NSArray *)items;`

- UIGravityBehavior常见方法
    - 添加1个物理仿真元素
    `- (void)addItem:(id <UIDynamicItem>)item;`
    // 移除1个物理仿真元素
    `- (void)removeItem:(id <UIDynamicItem>)item;`
- UIGravityBehavior常见属性
    - 添加到重力行为中的所有物理仿真元素
    `@property (nonatomic, readonly, copy) NSArray* items;`
    - 重力方向（是一个二维向量）
    `@property (readwrite, nonatomic) CGVector gravityDirection;`
    - 重力方向（是一个角度，以x轴正方向为0°，顺时针正数，逆时针负数）
    `@property (readwrite, nonatomic) CGFloat angle;`
    - 量级（用来控制加速度，1.0代表加速度是1000 points /second²）
    `@property (readwrite, nonatomic) CGFloat magnitude;`

```objc
/**
 *	重力
 */
- (void)testGravity
{
    // 创建重力仿真器
    UIGravityBehavior *gravity = [[UIGravityBehavior alloc] init];
    // 添加到指定对象
    [gravity addItem:self.greenView];
    // 重力方向
    gravity.gravityDirection = CGVectorMake(100, 100);
    // 重力加速度
    gravity.magnitude = 10;
    // 添加 物理仿真行为 到 物理仿真器 中, 开始物理仿真
    [self.anim addBehavior:gravity];
}
```
## 碰撞

- UICollisionBehavior边界相关的方法

```objc
- (void)addBoundaryWithIdentifier:(id <NSCopying>)identifier forPath:(UIBezierPath*)bezierPath;
- (void)addBoundaryWithIdentifier:(id <NSCopying>)identifier fromPoint:(CGPoint)p1 toPoint:(CGPoint)p2;
- (UIBezierPath*)boundaryWithIdentifier:(id <NSCopying>)identifier;
- (void)removeBoundaryWithIdentifier:(id <NSCopying>)identifier;
@property (nonatomic, readonly, copy) NSArray* boundaryIdentifiers;
- (void)removeAllBoundaries;
```
- UICollisionBehavior常见用法
    - 是否以参照视图的bounds为边界
    `@property (nonatomic, readwrite) BOOL translatesReferenceBoundsIntoBoundary;`
    - 设置参照视图的bounds为边界，并且设置内边距
    `- (void)setTranslatesReferenceBoundsIntoBoundaryWithInsets:(UIEdgeInsets)insets;`
    - 碰撞模式（分为3种，元素碰撞、边界碰撞、全体碰撞）
    `@property (nonatomic, readwrite) UICollisionBehaviorMode collisionMode;`
    - 代理对象（可以监听元素的碰撞过程）
    `@property (nonatomic, assign, readwrite) id <UICollisionBehaviorDelegate> collisionDelegate;`

```objc
/**
 *	碰撞
 */
- (void)testCollision1
{
    // 创建碰撞器
    UICollisionBehavior *collision = [[UICollisionBehavior alloc] init];
    // 让参照视图的bounds变为碰撞检测的边框
    collision.translatesReferenceBoundsIntoBoundary = YES;
    // 添加碰撞器到指定对象
    [collision addItem:self.greenView];
    // 创建物理仿真行为 - 重力行为
    UIGravityBehavior *gravity = [[UIGravityBehavior alloc] init];
    [gravity addItem:self.greenView];
    // 重力加速度
    gravity.magnitude = 10;
    // 添加 物理仿真行为 到 物理仿真器 中, 开始物理仿真
    [self.anim addBehavior:gravity];
    [self.anim addBehavior:collision];
}

- (void)testCollision2
{
    // 创建碰撞器
    UICollisionBehavior *collosion = [[UICollisionBehavior alloc] init];
    [collosion addItem:self.greenView];
    // 添加曲线
    UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0, 200, 375, 375)];
    [collosion addBoundaryWithIdentifier:@"bezier" forPath:path];

    // 添加边界,一条直线
    //    [collosion addBoundaryWithIdentifier:@"haha" fromPoint:CGPointMake(0, 300) toPoint:CGPointMake(200, 500)];

    // 创建物理仿真行为 - 重力行为
    UIGravityBehavior *gravity = [[UIGravityBehavior alloc] init];
    [gravity addItem:self.greenView];

    // 重力加速度
    gravity.magnitude = 10;
    // 添加 物理仿真行为 到 物理仿真器 中, 开始物理仿真
    [self.anim addBehavior:gravity];

    // 添加碰撞器行为
    [self.anim addBehavior:collosion];
}
```

## 吸附

- UISnapBehavior的初始化
    `- (instancetype)initWithItem:(id <UIDynamicItem>)item snapToPoint:(CGPoint)point;`

- UISnapBehavior常见属性
    - 用于减幅、减震（取值范围是0.0 ~ 1.0，值越大，震动幅度越小）
    `@property (nonatomic, assign) CGFloat damping;`
- UISnapBehavior使用注意
    - 如果要进行连续的捕捉行为，需要先把前面的捕捉行为从物理仿真器中移除

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    UITouch *touch = [touches anyObject];
    CGPoint point = [touch locationInView:self.view];

    // 吸附器
    UISnapBehavior *snap = [[UISnapBehavior alloc] initWithItem:self.greenView snapToPoint:point];
    // 吸附阻力
    snap.damping = 1.0;

    [self.anim removeAllBehaviors];
    [self.anim addBehavior:snap];
}
```
