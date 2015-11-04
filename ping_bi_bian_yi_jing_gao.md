# 屏蔽编译警告

```objc
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
    if ([self.selectedViewController respondsToSelector:@selector(isReadyForEditing)]) {
        boolNumber = [self.selectedViewController performSelector:@selector(isReadyForEditing)];
    }
#pragma clang diagnostic pop
```