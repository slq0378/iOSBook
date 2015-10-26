# 代理传值和block传值

## 代理传值和block传值

- 在两个不同的控制器之间传递数据，可以使用代理传值或者block传值。

- 例子是一个简单通讯录。

- 主界面如下：

![](../images/屏幕快照 2015-06-13 13.05.15.png)

- 添加联系人界面

![](../images/屏幕快照 2015-06-13 13.05.08.png)

- 查看/编辑联系人界面：默认是查看模式，点击编辑后进入编辑模式

![](../images/屏幕快照 2015-06-13 13.05.23.png)

- 编辑模式

![](../images/屏幕快照 2015-06-13 13.05.36.png)

- 数据更新成功。

![](../images/屏幕快照 2015-06-13 13.05.44.png)

- 其中添加联系人界面的数据传递使用代理方式实现。

- 编辑联系人界面的数据传递使用block实现。

- 下面来看具体过程

- 1、整个界面搭建

![](../images/屏幕快照 2015-06-13 13.32.34.png)

- 在storyboard里拖拽四个控制器，其中联系人界面是一个UITableView。界面之间的跳转使用代码实现，但是要给每一个控制器指定一个标识。按功能分别指定为login，contact，add,edit.

![](../images/屏幕快照 2015-06-13 13.25.46.png)

![](../images/屏幕快照 2015-06-13 13.37.10.png)

- 具体细节我就不说了，关键是代理传值的实现。

## 1、代理传值

- 数据传递方向是从add控制器传递到contact控制器。至于为什么使用代理，主要是为了降低类之间的耦合度。
- 1、这里需要给add控制器添加一个代理对象，然后定义一个代理需要遵守的协议。

```objc
    // 代理实现逆传数据
    @classSLQContact; // 模型对象
    @classSLQAddViewController; //
    @protocol SLQAddViewControllerDelegate<NSObject>

    @optional
    // 代理方法
    - (void)addViewController:(SLQAddViewController *)addVC DidClickBtnWithContact:(SLQContact *)contact;

    @end

    @interface SLQAddViewController : UIViewController
    /*代理对象*/
    @property (strong, nonatomic) id<SLQAddViewControllerDelegate> delegate;

    @end
```
- 2、等需要传递数据时只需通知代理一声即可，不需要关心代理如何做。
    - 在添加联系人界面里点击添加按钮后就去通知代理传递数据。

```objc
    // 添加联系人按钮，单击后传递数据到联系人控制器，并返回上一个界面
    - (IBAction)addBtn:(id)sender {

        //传递模型数据
        SLQContact *temp = [SLQContactcontactWithName:_nameField.textandPhone:_phoneField.text];

        // 通知代理
        if([_delegate respondsToSelector:@selector(addViewController:DidClickBtnWithContact:)])
        {
            [_delegateaddViewController:selfDidClickBtnWithContact:temp];
        }
        // 返回上一个界面
        [self.navigationControllerpopViewControllerAnimated:YES];
    }
```

- 3、还有最关键的一步，那就是指定代理对象是谁
    - 这里指定代理对象的是contact控制器，因为，要把数据传递给它，所以它作为接收者也就是代理方。

```objc
    // 添加按钮，按下后进入添加联系人界面
    - (IBAction)addBtn:(id)sender {
        //
    //    [self performSegueWithIdentifier:@"contactToAdd" sender:nil];
        // 通过代码获取storyboard中得控制器
        UIStoryboard *story = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
        // 获取main。storyboard中得控制器，以标识符区分
        SLQAddViewController* addVC = [story instantiateViewControllerWithIdentifier:@"add"];
        // 代理逆传数据
        //如果通过代码跳转的话，必须知道目的控制器，整个程序中也只有这个时候才知道下一个控制器是谁，所以在这里指定代理对象为contact控制器再好不过了
        addVC.delegate = self;
        // 跳转到添加联系人界面    [self.navigationController pushViewController:addVC animated:YES];
    }
    // 实现代理方法，记得让类遵守协议
    - (void)addViewController:(SLQAddViewController *)addVC DidClickBtnWithContact:(SLQContact *)contact
    {
        // 添加数据到数组
        [_contacts addObject:contact];

        // 刷洗表格
        [self.tableView reloadData];
    }
```

## 2、block传值

- 1、在编辑控制器中对block进行生声明以及定义

```objc
    // blocl传值使用
    @classSLQContact;
    // 声明block别名，参数为要传递的数据。
    typedef void(^SLQEditViewControllerBolok)(SLQContact *);

    @interface SLQEditViewController : UIViewController
    /*模型*/
    @property (strong, nonatomic) SLQContact *contact;
    /*block 对象*/
    @property (strong, nonatomic) SLQEditViewControllerBolok block;
    @end
```

- 2、在点击保存按钮后进行数据传递

```objc
    // 保存按钮事件
    - (IBAction)save:(id)sender
    {
        // 传递模型数据
        SLQContact *contact = [SLQContactcontactWithName:_nameField.textandPhone:_phoneField.text];
        // block实现传值，先检查是否有数据，如果有传递模型数据
        if(_block)
        {
            _block(contact);
        }
        // 回到上个界面
        [self.navigationControllerpopViewControllerAnimated:YES];
    }
```

- 3、关键一点还是要在数据接收方也就是contact控制器中对block内容进行包装
    -因为要通过代码跳转，同样需要知道目的控制器，跳转的地方就是选中cell的时候。

```objc
    // 选中cell后进入编辑界面
    - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
    {
        // 获取编辑控制器
        UIStoryboard *story  = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
        // 获取目的控制器
        SLQEditViewController *editVC = [story instantiateViewControllerWithIdentifier:@"edit"];
        // 传递模型数据到edit控制器（这是一种顺序传值）
        editVC.contact = self.contacts[indexPath.row];
        // 使用block包装之后要进行的操作
        editVC.block = ^(SLQContact *contact){
            // 修改数据
            [self.contacts replaceObjectAtIndex:indexPath.row withObject:contact];
            // 刷新表格
            [self.tableView reloadData];
        };
        // 跳转到编辑界面
        [self.navigationControllerpushViewController:editVC animated:YES];
    }
```

## 3、顺序传值
- 顺序传递数据比价简单，只需要接收方有一个属性对要传递的数据进行接收就行。
    - 上面在进入编辑控制界面时就要对数据进行传递，要把在联系人界面的数据传递到编辑控制器界面，然后对其进行修改。

```objc
    // 选中cell后进入编辑界面
    - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
    {
        // 获取编辑控制器
        UIStoryboard *story  = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
        // 获取目的控制器
        SLQEditViewController *editVC = [story instantiateViewControllerWithIdentifier:@"edit"];
        // 传递模型数据到edit控制器（这是一种顺序传值）
        editVC.contact = self.contacts[indexPath.row];
        // 使用block包装之后要进行的操作
        editVC.block = ^(SLQContact *contact){
            // 修改数据
            [self.contacts replaceObjectAtIndex:indexPath.row withObject:contact];
            // 刷新表格
            [self.tableView reloadData];
        };
        // 跳转到编辑界面
        [self.navigationControllerpushViewController:editVC animated:YES];
    }
```
- 上面的代码这样写。是因为在编辑控制器中已经定义了一个模型对象接收传递的数据。

```objc
    @interface SLQEditViewController : UIViewController
    /*模型对象，接收传递过来的数据*/
    @property (strong, nonatomic) SLQContact *contact;
    /*block 对象*/
    @property (strong, nonatomic) SLQEditViewControllerBolok block;
    @end
```
- 同样传递的地方也是在控制器跳转之前进行数据的传递。

## 4、总结

- 传值的方是由到导航控制器的行走方向决定的。
    -顺序传值：
        - 由源控制器传递当目的控制器。
        - 接收方有一个属性接收传递数据，在控制器跳转之前进行传递
    - 逆序传值：
        - 由源控制器传递当目的控制器。
    - 代理：
        - 在发送方对声明代理对象，然后定义代理协议,要传递的数据要放在代理方法的参数中，最后在触发事件后通知代理
        - 在接收方设置代理位接收方，实现代理方法
    - block：
        - 在发送方对声明block，要传递的数据要放在block的参数中，最后在触发事件后调用block
        - 在接收方设置block的内容

- 什么时候使用block？
    - 逆传:用block来传值,处理网络的时候经常使用block封装代码。
    - 请求网络数据（延迟）         先把展示到控件的代码先保存到block，等请求到数据的时候直接调用Block

http://www.cnblogs.com/songliquan/
