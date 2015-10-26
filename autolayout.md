#autolayout自动布局

## 在_storyboard/xib_文件中实现自动布局

- autolayout和frame属性是有冲突的，所以如果准备使用autolayout，就不要再代码中对控件的frame属性进行操作。

- 设置autolayout必须设置完全，必须包括位置信息和尺寸信息。也就是说必须有宽高和坐标位置,缺一不可。

- 简单示例：两个view（redView和blueView），等高等宽，redView距离左边和下边间距都是20，距离blueView间距也是20.blueView距离右边和下边都是20.
实现界面如下，每个view 都要添加六个约束，用来确定自己的位置和尺寸。

- 其中UILable比较特殊，它只需指定位置和宽度，高度系统会根据文字长度自动计算。
- 可以参看下面程序。
![](../images/autolayout01.png)

![](../images/autolayout02.png)



## 通过代码实现自动布局
- 代码实现Autolayout的步骤
	- 利用NSLayoutConstraint类创建具体的约束对象
	- 添加约束对象到相应的view上

```objc
	- (void)addConstraint:(NSLayoutConstraint *)constraint;
	- (void)addConstraints:(NSArray *)constraints;
```

- 代码实现Autolayout的注意点
	- 要先禁止autoresizing功能，设置view的下面属性为NO
```objc
    view.translatesAutoresizingMaskIntoConstraints = NO;
```
	- **添加约束之前，一定要保证相关控件都已经在各自的父控件上**
	- 不用再给view设置frame

- **一个NSLayoutConstraint对象就代表一个约束**
```objc
// 创建约束对象的常用方法
+(id)constraintWithItem:(id)view1
attribute:(NSLayoutAttribute)attr1
relatedBy:(NSLayoutRelation)relation
					toItem:(id)view2
attribute:(NSLayoutAttribute)attr2
	multiplier:(CGFloat)multiplier
				constant:(CGFloat)c;
				view1:要约束的控件
				attr1:约束的类型（做怎样的约束）
				relation:与参照控件之间的关系
				view2:参照的控件
				attr2:约束的类型（做怎样的约束）
				multiplier:乘数
				c:常量
```

- **自动布局的核心计算公式**
	**view1.attr1 =（view2.attr2 * multiplier）+ c ;**

- **约束添加规则**
	- 对于两个同层级view之间的约束关系，添加到它们的父view上。
	- 对于两个不同层级view之间的约束关系，添加到他们最近的共同父view上
	- 对于有层次关系的两个view之间的约束关系，添加到层次较高的父view上


### 简单练习:一个view
- 实现一个大小为100*200的redView显示到view中，距离左边和顶部间距都是20

```objc
    // 实现一个大小为100*200的redView显示到view中，距离左边和顶部间距都是20
    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    // 关闭控件的自动添加约束功能
    redView.translatesAutoresizingMaskIntoConstraints = NO;
    // 先把控件添加到父控件才能接着添加约束
    [self.view addSubview:redView];
    // 设置宽度
    NSLayoutConstraint *widthConstraint = [NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeWidth multiplier:0 constant:100];
    [redView addConstraint:widthConstraint];
    // 设置高度
    NSLayoutConstraint *heightConstraint = [NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeHeight multiplier:0 constant:200];
    [redView addConstraint:heightConstraint];
    // 设置尺寸:距离顶部20
    NSLayoutConstraint *topConstraint = [NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTop multiplier:1 constant:20];
    [self.view addConstraint:topConstraint];
    // 距离左边约束20
    NSLayoutConstraint *leftConstraint = [NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeLeft multiplier:1 constant:20];
    [self.view addConstraint:leftConstraint];

```
###练习：两个view

- 在控制器view顶部添加2个view，1个蓝色，1个红色
	- 2个view高度永远相等
	- 红色view和蓝色view右边对齐
	- 蓝色view距离父控件左边、右边、上边间距相等
	- 蓝色view距离红色view间距固定
	- 红色view的左边和父控件的中点对齐

```objc
	UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    // 关闭控件的自动添加约束功能
    redView.translatesAutoresizingMaskIntoConstraints = NO;
    // 先把控件添加到父控件才能接着添加约束
    [self.view addSubview:redView];

    UIView *blueView = [[UIView alloc] init];
    blueView.backgroundColor = [UIColor blueColor];
    // 关闭控件的自动添加约束功能
    blueView.translatesAutoresizingMaskIntoConstraints = NO;
    // 先把控件添加到父控件才能接着添加约束
    [self.view addSubview:blueView];

    /**
     *  - 在控制器view顶部添加2个view，1个蓝色，1个红色
     - 2个view高度永远相等
     - 红色view和蓝色view右边对齐
     - 蓝色view距离父控件左边、右边、上边间距相等 :20
     - 蓝色view距离红色view间距固定
     - 红色view的左边和父控件的中点对齐
     */

    /**************blue*********************************************************/
    // blueHeight
    NSLayoutConstraint *blueHeight = [NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:nil attribute:kNilOptions multiplier:0 constant:40];
    [self.view addConstraint:blueHeight]; // 同级控件之间的约束要添加到共同的父控件

    //蓝色view距离父控件左边、右边、上边间距相等 :20
    NSLayoutConstraint *blueViewRight = [NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeRight relatedBy:NSLayoutRelationEqual toItem:self.view  attribute:NSLayoutAttributeRight multiplier:1 constant:-20];
    [self.view addConstraint:blueViewRight];

    NSLayoutConstraint *blueViewTop = [NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:self.view  attribute:NSLayoutAttributeTop multiplier:1 constant:20];
    [self.view addConstraint:blueViewTop];

    NSLayoutConstraint *blueViewLeft = [NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:self.view  attribute:NSLayoutAttributeLeft multiplier:1 constant:20];
    [self.view addConstraint:blueViewLeft];

    /**************red*********************************************************/
    //  - redview与blueview等高
    NSLayoutConstraint *equalHeight = [NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem: blueView attribute:NSLayoutAttributeHeight multiplier:1 constant:0];
    [self.view addConstraint:equalHeight]; //

    //   - redview的左边和父控件的中点对齐
    NSLayoutConstraint *centerX = [NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeCenterX multiplier:1 constant:0];
    [self.view addConstraint:centerX]; //

    //   - redview距离blueView间距20
    NSLayoutConstraint *margin = [NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeBottom multiplier:1 constant:20];
    [self.view addConstraint:margin]; //

       //- 红色view和蓝色view右边对齐
    NSLayoutConstraint *equalRight = [NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeRight relatedBy:NSLayoutRelationEqual toItem:blueView  attribute:NSLayoutAttributeRight multiplier:1 constant:0];
    [self.view addConstraint:equalRight]; //

```

### visual Format Lauguage ,可视化格式语言

可以使用VFL实现自动布局代码的简化。

#### VFL的使用
- 使用VFL来创建约束数组

```objc
+ (NSArray *)constraintsWithVisualFormat:(NSString *)format options:(NSLayoutFormatOptions)opts metrics:(NSDictionary *)metrics views:(NSDictionary *)views;
format ：VFL语句
opts ：约束类型
metrics ：VFL语句中用到的具体数值
views ：VFL语句中用到的控件
```

- 创建一个字典（内部包含VFL语句中用到的控件）的快捷宏定义
```objc
    NSDictionaryOfVariableBindings(...)
	NSDictionaryOfVariableBindings(abc);
```
返回一个字典对象，内容为{@"abc" : abc};

- 代码示例

```objc

    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    // 关闭控件的自动添加约束功能
    redView.translatesAutoresizingMaskIntoConstraints = NO;
    // 先把控件添加到父控件才能接着添加约束
    [self.view addSubview:redView];
    NSString *vfl  = @"H:[redView(100)]-20-|"; // 设置宽度100，距离右边20
    NSDictionary *view = @{@"redView" : redView};
    // 水平方向约束
    NSArray *constraint = [NSLayoutConstraint constraintsWithVisualFormat:vfl options:kNilOptions metrics:nil views:view];
    [self.view addConstraints:constraint];
    // 垂直方向约束
    vfl = @"V:|-100-[redView(200)]"; // 设置高度200，距离顶部100
    NSArray *constraint2 = [NSLayoutConstraint constraintsWithVisualFormat:vfl options:kNilOptions metrics:nil views:view];
    [self.view addConstraints:constraint2 ];

```

-  居中显示

```objc
    // 居中显示
    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    // 关闭控件的自动添加约束功能
    redView.translatesAutoresizingMaskIntoConstraints = NO;
    // 先把控件添加到父控件才能接着添加约束
    [self.view addSubview:redView];

    NSNumber *margin = @100;

    NSString *vfl  = @"H:|-margin-[redView]-margin-|"; // 设置宽度100，距离右边20
    NSDictionary *view = @{@"redView" : redView};
    NSDictionary *metric = @{@"margin" : margin};
    // 水平方向约束
    NSArray *constraint = [NSLayoutConstraint constraintsWithVisualFormat:vfl options:NSLayoutFormatAlignAllCenterX metrics:metric views:view];
    [self.view addConstraints:constraint];

    // 垂直方向约束
    NSDictionary *view2 = NSDictionaryOfVariableBindings(redView);
    NSDictionary *metric2 = NSDictionaryOfVariableBindings(margin);
    vfl = @"V:|-margin-[redView]-margin-|"; // 设置高度200，距离顶部100
    NSArray *constraint2 = [NSLayoutConstraint constraintsWithVisualFormat:vfl options:NSLayoutFormatAlignAllCenterY metrics:metric2 views:view2];
    [self.view addConstraints:constraint2 ];
```



## 使用代码实现Autolayout的方法3 - Masonry
- 使用步骤
    - 添加Masonry文件夹的所有源代码到项目中
    - 添加2个宏、导入主头文件
    ```objc
    // 只要添加了这个宏，就不用带mas_前缀
    #define MAS_SHORTHAND
// 只要添加了这个宏，equalTo就等价于mas_equalTo
#define MAS_SHORTHAND_GLOBALS
// 这个头文件一定要放在上面两个宏的后面
#import "Masonry.h"
    ```

- 添加约束的方法

```objc
// 这个方法只会添加新的约束
 [view makeConstraints:^(MASConstraintMaker *make) {

 }];

// 这个方法会将以前的所有约束删掉，添加新的约束
 [view remakeConstraints:^(MASConstraintMaker *make) {

 }];

 // 这个方法将会覆盖以前的某些特定的约束
 [view updateConstraints:^(MASConstraintMaker *make) {

 }];
```

- 约束的类型
```objc
1.尺寸：width\height\size
2.边界：left\leading\right\trailing\top\bottom
3.中心点：center\centerX\centerY
4.边界：edges
```
- 示例代码1
```objc
    // 居中显示
    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    [self.view addSubview:redView];

    // 可以使用三个方法来添加约束。
    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerX.equalTo(self.view);
        make.centerY.equalTo(self.view);
        make.height.equalTo(100);
        make.width.equalTo(200);
    }];
```
- 示例代码2
```objc
    //并排位于底部，间距20

    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    [self.view addSubview:redView];

    UIView *blueView= [[UIView alloc] init];
    blueView.backgroundColor = [UIColor blueColor];
    [self.view addSubview:blueView];

    // 添加约束
    [redView makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view.left).offset(20); // 左边间距20
        make.right.equalTo(blueView.left).offset(-20); // 右边和blueView间距20
        make.bottom.equalTo(self.view.bottom).offset(-20); // 底部间距20

        make.height.equalTo(200); // 高度200

    }];

    [blueView makeConstraints:^(MASConstraintMaker *make) {
        make.right.equalTo(self.view.right).offset(-20); // 右边间距20
        make.bottom.equalTo(redView.bottom); // 和redView底部间距相同

        make.height.equalTo(redView); // 等宽
        make.width.equalTo(redView); // 等高
    }];
```
- 示例代码3

```objc
    //四个View，平分整个屏幕
    //红色
    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    [self.view addSubview:redView];
    // 蓝色
    UIView *blueView= [[UIView alloc] init];
    blueView.backgroundColor = [UIColor blueColor];
    [self.view addSubview:blueView];
    // 黑色
    UIView *blackView = [[UIView alloc] init];
    blackView.backgroundColor = [UIColor blackColor];
    [self.view addSubview:blackView];
    // 绿色
    UIView *greenView= [[UIView alloc] init];
    greenView.backgroundColor = [UIColor greenColor];
    [self.view addSubview:greenView];


    // autolayout
    [redView makeConstraints:^(MASConstraintMaker *make) {
        make.left.and.top.equalTo(self.view);
        make.right.equalTo(self.view.centerX);
        make.bottom.equalTo(self.view.centerY);
    }];

    [blueView makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(redView.right);
        make.right.equalTo(self.view);
        make.height.equalTo(redView);

    }];

    [blackView makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(redView.bottom);
        make.height.equalTo(redView);
        make.width.equalTo(redView);
    }];

    [greenView makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(blueView.bottom);
        make.left.equalTo(blackView.right);
        make.height.equalTo(blueView);
        make.width.equalTo(blueView);
    }];

    // 红色View内部
    UIImageView *image = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"010.png"]];
    UILabel *name = [[UILabel alloc] init];
    name.text = @"红色";
    name.textAlignment = NSTextAlignmentCenter;
    name.backgroundColor = [UIColor purpleColor];
    [redView addSubview:image];
    [redView addSubview:name];
    [image makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(redView.center).offset(-20);
    }];
    [name makeConstraints:^(MASConstraintMaker *make) {
        make.left.right.equalTo(redView.left);
        make.bottom.equalTo(redView.bottom);
        make.height.equalTo(40);
    }];
```

- 代码示例4

```objc

    // masonry 自动布局
    UIView *redView = [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    [self.view addSubview:redView];

    UIView *blueView= [[UIView alloc] init];
    blueView.backgroundColor = [UIColor blueColor];
    [self.view addSubview:blueView];


    [redView makeConstraints:^(MASConstraintMaker *make) {
        // 大小100*100，居中显示
        //make.size.equalTo(100);
        //make.center.equalTo(self.view);

        //居中显示，直接设置距离四面的距离
        UIEdgeInsets eda = {100,100,100,100};
        make.edges.insets(eda);
        //
    }];

    // blueView位于redView中间
    [blueView makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(redView);
        make.height.equalTo(redView.height).multipliedBy(0.5); // 乘法
        make.width.equalTo(redView.width).dividedBy(3).offset(20); // 除法+偏移量
    }];

```












