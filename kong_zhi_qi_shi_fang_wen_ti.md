# iOS中控制器的释放问题

- ARC工程是可以重写dealloc方法并被系统调用的，但不需要手动调用父类的dealloc，手写[super dealloc]方法会报错，事实上系统会自动帮你调用父类的dealloc方法，不需要你实现。可以通过在dealloc方法中打印log查看控制器是否被释放。

- 控制器在被pop后移出栈后会被释放，但有些时候会发现控制器出栈的时候不会调用dealloc方法，归根结底，是因为当前控制器被某个对象强引用了，控制器的引用计数不为0，系统无法帮你释放这部分内存。

## 控制器中NSTimer没有被销毁

- 当控制器中存在NSTimer时，就需要注意，因为当[NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(updateTime:) userInfo:nil repeats:YES];时，这个/target:self/ 就增加了VC的RetarnCountr, 如果你不将这个timer invalidate，就别想调用dealloc。需要在viewWillDisappear之前需要把控制器用到的NSTimer销毁。

```objc
     [timer invalidate]; // 销毁timer
     timer = nil; // 置nil
```
## 控制器中的代理不是weak属性

- `例如@property (nonatomic, weak) id<SLQPopViewDelegate>` 
- delegate;代理要使用弱引用，因为自定义控件是加载在视图控制器中的，视图控制器view对自定义控件是强引用，如果代理属性设置为strong，则意味着delegate对视图控制器也进行了强引用，会造成循环引用。导致控制器无法被释放，最终导致内存泄漏。

## 控制器中block的循环引用

- block会把它里面的所有对象强引用(在ARC下)/PS:MRC下会retain加1/，包括当前控制器self，因此有可能会出现循环引用的问题。
- 即一个对象有一个Block属性，然而这个Block属性中又引用了对象的其他成员变量，那么就会对这个变量本身产生强应用，那么这个对象本身和他自己的Block属性就形成了循环引用。在ARC下需要修改成这样：(/也就是生成一个对自身对象的弱引用/)

- `__weak typeof(self) weakSelf = self;`
- 即：保险起见block中所有的涉及到self的全给替换成weakSelf