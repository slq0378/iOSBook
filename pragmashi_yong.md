# pragma使用

## 组织代码

- 组织N个方法为一个section的依据是什么呢？这个就见仁见智了。一般来说：
    - 将一个protocol的方法组织成一个section；
    - 将target-action类型方法组织成一个section；
    - 将notification相关方法组织成一个section；
    - 将需要override的父类方法组成成一个section；

## 屏蔽警告

```objc
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
    if ([self.selectedViewController respondsToSelector:@selector(isReadyForEditing)]) {
        boolNumber = [self.selectedViewController performSelector:@selector(isReadyForEditing)];
    }
#pragma clang diagnostic pop
```

- 某些时候如果我们的源码在编译过程中出现大量的编译警告时，看起来是挺不爽的；但又确实没办法解决警告问题的时候，我们可以使用下面的方法来屏蔽指定的某个文件的所有警告信息。
    - 1、在Xcode中选中工程文件。
    - 2、在右边面板中选中“Build Phases”。
    - 3、展开“Compile Sources”
    - 4、在需要屏蔽警告的源文件一行中双击“Compiler F lags”。
    - 
    - 5、在弹出窗口中输入-w



然后再编译一下看看，警告是否没有了呢，呵呵。