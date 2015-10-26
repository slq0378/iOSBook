# 通讯录

- 在iOS中，有2个框架可以访问用户的通讯录

- AddressBookUI.framework
	- 提供了联系人列表界面、联系人详情界面、添加联系人界面等
	- 一般用于选择联系人
- AddressBook.framework
	- 纯C语言的API，仅仅是获得联系人数据
	- 没有提供UI界面展示，需要自己搭建联系人展示界面
	- 里面的数据类型大部分基于Core Foundation框架，使用起来极其蛋疼
- 从iOS6开始，需要得到用户的授权才能访问通讯录，因此在使用之前，需要检查用户是否已经授权
	- 获得通讯录的授权状态：ABAddressBookGetAuthorizationStatus()
- 各种状态
	- kABAuthorizationStatusNotDetermined
		- 用户还没有决定是否授权你的程序进行访问
	- kABAuthorizationStatusRestricted
		- iOS设备上一些许可配置阻止程序与通讯录数据库进行交互
	- kABAuthorizationStatusDenied
		- 用户明确的拒绝了你的程序对通讯录的访问
	- kABAuthorizationStatusAuthorized
		- 用户已经授权给你的程序对通讯录进行访问

## AddressBookUI

### 联系人简介

- 所有的属性常量值都定义在了ABPerson.h头文件中
- 联系人属性包括以下类型：
	- 简单属性：姓、名等 `CFStringRef`
	- 多重属性：电话号码、电子邮件等 `ABMultiValueRef`
	- 组合属性：地址等
- 注意：使用ABRecordCopyValue可以从一条Person记录中获取到对应的记录，但是后续处理则需要根据记录的具体类型加以区分


### 实现过程

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{

    // 1、创建选择联系人的界面
    ABPeoplePickerNavigationController *people = [[ABPeoplePickerNavigationController alloc] init];

    // 2、设置代理
    people.peoplePickerDelegate = self;

    // 3、弹出界面
    [self presentViewController:people animated:YES completion:nil];
}
```

- 代理方法 `ABPeoplePickerNavigationController`

```objc
#pragma mark - ABPeoplePickerNavigationControllerDelegate
// 选中某一个联系人的时候,会执行该代理方法
// 如果实现了该方法,那么就不会进入联系人的详细界面
- (void)peoplePickerNavigationController:(ABPeoplePickerNavigationController *)peoplePicker didSelectPerson:(ABRecordRef)person
{
    // 获取联系人信息
    // 1、获取联系人姓名
    CFStringRef firstname = ABRecordCopyValue(person, kABPersonFirstNameProperty);
    CFStringRef lastname = ABRecordCopyValue(person, kABPersonLastNameProperty);
    /*
     __bridge NSString * : 将CoreFoundation框架的对象所有权交给Foundation框架来使用,但是Foundation框架中的对象并不能管理该对象内存
     __bridge_transfer NSString * : 将CoreFoundation框架的对象所有权交给Foundation来管理,如果Foundation中对象销毁,那么我们之前的对象(CoreFoundation)会一起销毁


    // 这样转换有内存泄漏
//    NSString *firstName = (__bridge NSString *)(firstname);
//    NSString *lastName = (__bridge NSString *)(lastname);
     */
    NSString *firstName = (__bridge_transfer NSString *)(firstname);
    NSString *lastName = (__bridge_transfer NSString *)(lastname);
    NSLog(@"%@---%@",firstName,lastName);

    // 2、获取联系人号码：组合数据
    ABMultiValueRef phones = ABRecordCopyValue(person,kABPersonPhoneProperty);
    CFIndex count = ABMultiValueGetCount(phones);
    for (CFIndex i = 0; i < count; i ++) {
        // 电话标签
        NSString *phoneLabel =  (__bridge_transfer NSString *)ABMultiValueCopyLabelAtIndex(phones, i);
        // 电话号码
        NSString *phoneValue =  (__bridge_transfer NSString *)ABMultiValueCopyValueAtIndex(phones, i);
        NSLog(@"%@-%@",phoneLabel,phoneValue);
    }
    // 3.释放不再使用的对象
    CFRelease(phones);
}

// 选择某一个联系人的某一个属性时,会执行该方法
// property选中的属性
// identifier : 每一个属性都由一个对应标示
// 如果实现了该方法,那么选中一个联系人的属性时,就会推出控制器.不会进入下一个页面
- (void)peoplePickerNavigationController:(ABPeoplePickerNavigationController *)peoplePicker didSelectPerson:(ABRecordRef)person property:(ABPropertyID)property identifier:(ABMultiValueIdentifier)identifier
{
    NSLog(@"选中联系人");
}

// 点击取消按钮进这个方法
- (void)peoplePickerNavigationControllerDidCancel:(ABPeoplePickerNavigationController *)peoplePicker
{
    NSLog(@"退出联系人");
}
```

### 编辑联系人

- 添加联系人的步骤
    - 通过ABPersonCreate函数创建一个新的联系人（返回ABRecordRef）
    - 通过ABRecordSetValue函数设置联系人的属性
    - 通过ABAddressBookAddRecord函数将联系人添加到通讯录数据库中
    - 通过ABAddressBookSave函数保存刚才所作的修改

- 可以通过ABAddressBookHasUnsavedChanges函数判断是否有未保存的修改
- 当决定是否更改通讯录数据库后,你可以分别使用 AbAddressBookSave 或 ABAddressBookRevert 方式来保存或放弃更改

- 添加群组的步骤大体和添加联系人一致
    - 通过ABPersonCreate函数创建一个新的组（返回ABRecordRef）
    - 通过ABRecordSetValue函数设置组名
    - 通过ABAddressBookAddRecord函数将组添加到通讯录数据库中
    - 通过ABAddressBookSave函数保存刚才所作的修改

- 想操作联系人的头像，有以下函数
    - BPersonHasImageData //判断通讯录中的联系人是否有图片
    - ABPersonCopyImageData // 取得图片数据(假如有的话)
    - ABPersonSetImageData //设置联系人的图片数据

## 桥接方式

- `__bridge NSString * `: 将CoreFoundation框架的对象所有权交给Foundation框架来使用,但是Foundation框架中的对象并不能管理该对象内存
- `__bridge_transfer NSString *` :将CoreFoundation框架的对象所有权交给Foundation来管理,如果Foundation中对象销毁,那么我们之前的对象(CoreFoundation)会一起销毁

```objc
    // 获取联系人信息
    CFStringRef firstname = ABRecordCopyValue(person, kABPersonFirstNameProperty);
    CFStringRef lastname = ABRecordCopyValue(person, kABPersonLastNameProperty);

    // 这样转换有内存泄漏
//    NSString *firstName = (__bridge NSString *)(firstname);
//    NSString *lastName = (__bridge NSString *)(lastname);
    NSString *firstName = (__bridge_transfer NSString *)(firstname);
    NSString *lastName = (__bridge_transfer NSString *)(lastname);
    NSLog(@"%@---%@",firstName,lastName);
```

## AddressBook

- 请求授权 appDelegate

```objc
    // 请求获取通讯录

    // 1.获取授权的状态
    ABAuthorizationStatus status = ABAddressBookGetAuthorizationStatus();
    // 2.判断授权状态,如果是未决定状态,才需要请求
    if (status == kABAuthorizationStatusNotDetermined) {
        // 2.1.创建通信录对象
        ABAddressBookRef book = ABAddressBookCreateWithOptions(NULL, NULL);
        // 2.2.请求授权
        ABAddressBookRequestAccessWithCompletion(book, ^(bool granted, CFErrorRef error) {
            if (granted) {
                NSLog(@"授权成功");
            }
            else
            {
                 NSLog(@"授权失败");
            }
        });
    }
```
- 获取联系人列表，只能一次性获取全部

```objc
// 1.获取授权的状态
ABAuthorizationStatus status = ABAddressBookGetAuthorizationStatus();

// 2.判断授权状态,如果是未决定状态,直接退出
if (status == kABAuthorizationStatusNotDetermined) {
    return;
}

// 3.创建通信录对象
ABAddressBookRef book = ABAddressBookCreateWithOptions(NULL, NULL);

// 4.从通信录对象中,将所有的联系人拷贝出来
CFArrayRef peoples = ABAddressBookCopyArrayOfAllPeople(book);

// 5.遍历所有的联系人(每一个联系人都是一条记录)
CFIndex count = CFArrayGetCount(peoples);
for (CFIndex i = 0 ; i < count; i ++) {

    // 6.获取到联系人
    ABRecordRef person = CFArrayGetValueAtIndex(peoples, i);

    // 7.获取姓名
    NSString *firstName = (__bridge_transfer NSString *)(ABRecordCopyValue(person, kABPersonFirstNameProperty));
    NSString * lastName = (__bridge_transfer NSString *)(ABRecordCopyValue(person, kABPersonLastNameProperty));
    // 获取手机号
    NSString *phoneNumber = (__bridge_transfer NSString *)ABRecordCopyValue(person, kABPersonPhoneProperty);
    NSLog(@"%@  %@  %@",firstName,lastName,phoneNumber);
}
// 8、释放所有对象
CFRelease(book);
CFRelease(peoples);
```


## 第三方框架
- `RHAddressBook`
- 获取授权

```objc
    // 1.获取授权的状态
    ABAuthorizationStatus status = ABAddressBookGetAuthorizationStatus();
    // 2.判断授权状态,如果是未决定状态,才需要请求
    if (status == kABAuthorizationStatusNotDetermined) {
        // 2.1.创建通信录对象
        RHAddressBook *book = [[RHAddressBook alloc] init];
        // 2.2.请求授权
        [book requestAuthorizationWithCompletion:^(bool granted, NSError *error) {
            if (granted) {
                NSLog(@"授权成功");
            }
            else
            {
                NSLog(@"授权失败");
            }
        }];
    }
```

- 加载信息
```objc
// 1.获取授权的状态
RHAuthorizationStatus status = [RHAddressBook authorizationStatus];

// 2.判断授权状态,如果是未决定状态,直接退出
if (status != RHAuthorizationStatusAuthorized ) {
    return;
}

// 3.创建通信录对象
RHAddressBook *book = [[RHAddressBook alloc] init];

// 4.从通信录对象中,将所有的联系人拷贝出来
NSArray *peoples = book.people;

// 5.遍历所有的联系人(每一个联系人都是一条记录)
for (RHPerson *person in peoples) {

    // 6.获取到联系人
    NSLog(@"%@  %@",person.firstName,person.lastName);
    // 获取手机号
    RHMultiValue *phones = person.phoneNumbers;
    for (NSInteger i = 0 ; i < phones.count ; i ++) {
        NSString *label = [phones labelAtIndex:i];
        NSString *value = [phones labelAtIndex:i];
        NSLog(@"%@  %@",label,value);
    }
}
```




