# UIMenuController的示例

![](/images/屏幕快照 2015-08-07 13.49.31.png)


## UIMenuController须知
- 默认情况下, 有以下控件已经支持UIMenuController
    - UITextField
    - UITextView
    - UIWebView

## 让其他控件也支持UIMenuController(比如UILabel)
- 自定义UILabel
- 重写2个方法

```objc
/**
 * 让label有资格成为第一响应者
 */
- (BOOL)canBecomeFirstResponder
{
    return YES;
}

/**
 * label能执行哪些操作(比如copy, paste等等)
 * @return  YES:支持这种操作
 */
- (BOOL)canPerformAction:(SEL)action withSender:(id)sender
{
    if (action == @selector(cut:) || action == @selector(copy:) || action == @selector(paste:)) return YES;

    return NO;
}
```

- 实现各种操作方法

```objc
- (void)cut:(UIMenuController *)menu
{
    // 将自己的文字复制到粘贴板
    [self copy:menu];

    // 清空文字
    self.text = nil;
}

- (void)copy:(UIMenuController *)menu
{
    // 将自己的文字复制到粘贴板
    UIPasteboard *board = [UIPasteboard generalPasteboard];
    board.string = self.text;
}

- (void)paste:(UIMenuController *)menu
{
    // 将粘贴板的文字 复制 到自己身上
    UIPasteboard *board = [UIPasteboard generalPasteboard];
    self.text = board.string;
}
```
- 让label成为第一响应者

```objc
// 这里的self是label
[self becomeFirstResponder];
```

- 显示UIMenuController

```objc
UIMenuController *menu = [UIMenuController sharedMenuController];
// targetRect: MenuController需要指向的矩形框
// targetView: targetRect会以targetView的左上角为坐标原点
// 设置menu显示区域，一下两种效果一样，主要是确定menu的显示位置
// [menu setTargetRect:self.bounds inView:self];
    [menu setTargetRect:self.frame inView:self.superview];
[menu setTargetRect:self.bounds inView:self];
// [menu setTargetRect:self.frame inView:self.superview];
[menu setMenuVisible:YES animated:YES];
```

## 自定义UIMenuController内部的Item
- 添加item
    - 点击item, 默认会调用控制器的方法

```objc
// 添加MenuItem(点击item, 默认会调用控制器的方法)
UIMenuItem *ding = [[UIMenuItem alloc] initWithTitle:@"顶" action:@selector(ding:)];
UIMenuItem *replay = [[UIMenuItem alloc] initWithTitle:@"回复" action:@selector(replay:)];
UIMenuItem *report = [[UIMenuItem alloc] initWithTitle:@"举报" action:@selector(report:)];
menu.menuItems = @[ding, replay, report];
```
## 在tableView控制器的cell中显示

![](/images/屏幕快照 2015-08-07 16.37.14.png)

- 首先在选中cell时做出反应
```objc
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView
{
    // 隐藏键盘
    [self.view endEditing:YES];
    // 隐藏menu
    [[UIMenuController sharedMenuController] setMenuVisible:NO animated:YES];
}
// 选中cell弹出menu
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{

    UIMenuController *menu = [UIMenuController sharedMenuController];
    // 如果没有隐藏菜单控制器，就隐藏
    if (menu.isMenuVisible ) {
        [menu setMenuVisible:NO animated:YES];
    }
    else // 否则就显示出来
    {
        // 获取选中的cell
        SLQCommentCell *cell = (SLQCommentCell *)[tableView cellForRowAtIndexPath:indexPath];
        [cell becomeFirstResponder];

        // 添加自定义菜单
        UIMenuItem *ding = [[UIMenuItem alloc] initWithTitle:@"顶" action:@selector(ding:)];
        UIMenuItem *repost= [[UIMenuItem alloc] initWithTitle:@"回复" action:@selector(repost:)];
        UIMenuItem *report = [[UIMenuItem alloc] initWithTitle:@"举报" action:@selector(report:)];

        menu.menuItems = @[ding,repost,report];

        [menu setTargetRect:cell.bounds inView:cell];
        NSLog(@"%@", NSStringFromCGRect(cell.bounds));
        // 显示菜单
        [menu setMenuVisible:YES animated:YES];
    }

}

#pragma mark - MenuItem方法实现
- (void)ding:(UIMenuItem *)menu
{
    SLQLogFunction;
}
- (void)repost:(UIMenuItem *)menu
{
    SLQLogFunction;
}
- (void)report:(UIMenuItem *)menu
{
    SLQLogFunction;
}
```
- 其次在cell模型中实现方法

```objc
/**
 *	关闭系统默认menu
 */
- (BOOL)canPerformAction:(SEL)action withSender:(id)sender {
    return  NO;
}
/**
 *	让cell可以成为第一响应者
 */
- (BOOL)canBecomeFirstResponder {
    return  YES;
}
```



