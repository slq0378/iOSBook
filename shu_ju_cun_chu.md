# 数据存储

-  IOS数据存储的几种方式
    - XML属性列表（plist）
    - 归档 Preference(偏好设置)
    - NSKeyedArchiver归档(NSCoding)
    - SQLite3
    - Core Data

## 沙盒
- 每个iOS应用都有自己的应用沙盒(应用沙盒就是文件系统目录)，与其他文件系统隔离。应用必须待在自己的沙盒里，其他应用不能访问该沙盒。沙盒的结构如下所示：

![](../images/屏幕快照 2015-06-13 19.05.06.png)

![](../images/屏幕快照 2015-06-13 19.08.29.png)

- 分类
    - 应用程序包：(上图中的QQ.app)包含了所有的资源文件和可执行文件。
    - Documents：保存应用运行时生成的需要持久化的数据，iTunes同步设备时会备份该目录。例如，游戏应用可将游戏存档保存在该目录。
    - tmp：保存应用运行时所需的临时数据，使用完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录。
    - Library/Caches：保存应用运行时生成的需要持久化的数据，iTunes同步设备时不会备份该目录。一般存储体积大、不需要备份的非重要数据。
    - Library/Preference：保存应用的所有偏好设置，iOS的Settings(设置)应用会在该目录中查找应用的设置信息。iTunes同步设备时会备份该目录。

- 每个目录的获取方法

```objc
// 临时目录
NSString *path = NSTemporaryDirectory();
// /Users/song/Library/Developer/CoreSimulator/Devices/B040D34E-CF0E-41C2-9BD0-C309566CC349/data/Containers/Data/Application/A8519BA5-FB34-4B31-BB72-3948264AFB37/tmp/
// 主目录
NSString *path = NSHomeDirectory();
//  /Users/song/Library/Developer/CoreSimulator/Devices/B040D34E-CF0E-41C2-9BD0-C309566CC349/data/Containers/Data/Application/E6A8A443-55CA-463B-886C-DAAF69E8F3BC

// Documents 目录，第三个参数为yes显示全路径，为NO时显示位波浪线 “~”
NSString *path1 = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
// /Users/song/Library/Developer/CoreSimulator/Devices/B040D34E-CF0E-41C2-9BD0-C309566CC349/data/Containers/Data/Application/61B033AA-BBFB-4A3A-AD6E-2E332B78C2ED/Documents
NSString *path1 = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, NO)[0];
//  ~/Documents
// cache 目录
NSString *path2 =NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
// /Users/song/Library/Developer/CoreSimulator/Devices/B040D34E-CF0E-41C2-9BD0-C309566CC349/data/Containers/Data/Application/1B4923AF-4B1F-4C7A-941E-D3621D7945C3/Library/Caches
// PreferencePanes 目录
NSString *path2 =NSSearchPathForDirectoriesInDomains(NSPreferencePanesDirectory, NSUserDomainMask, YES)[0];
// /Users/song/Library/Developer/CoreSimulator/Devices/B040D34E-CF0E-41C2-9BD0-C309566CC349/data/Containers/Data/Application/D654173E-DC23-4951-8C28-016878B5246B/Library/PreferencePanes
```

- 还有一个特殊的单例对象用来保存用户设置

```objc
    // 默认保存到PreferencePanes目录
    [[NSUserDefaults standardUserDefaults] setObject:@“hah” forKey:@"1"];
    [[NSUserDefaults standardUserDefaults] setBool:YES forKey:@“2”];
```

## 1、plist文件

- 属性列表是一种XML格式的文件，拓展名为plist

- 如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，就可以使用writeToFile:atomically:方法直接将对象写到属性列表文件中.

- 保存数据

```objc
    // plist保存数据
    // 保存用户名、密码、记住密码、自动登录
    NSMutableDictionary *dict = [NSMutableDictionarydictionary];
    [dict setObject:_userField.text forKey:@"user"];
    [dict setObject:_pwdField.text forKey:@"pwd"];
    [dict setObject:@(_rmbPwdSwitch.on) forKey:@"rmb"];
    [dict setObject:@(_autoLoginSwitch.on) forKey:@"auto"];
    // 获取cache路径
    NSString *cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
    // 取得文件的全路径
    NSString *filePath = [cachePath stringByAppendingPathComponent:@"userInfo.plist"];
    // 写入到文件
            [dict writeToFile:filePath atomically:YES];
```

![](../images/屏幕快照 2015-06-13 19.49.39.png)

![](../images/屏幕快照 2015-06-13 19.49.57.png)


- 读取数据

```objc
    // 读取plist中数据，显示在textField中
    // 获取cache路径
    NSString *cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
    // 取得文件的全路径
    NSString *filePath = [cachePath stringByAppendingPathComponent:@"userInfo.plist"];
    // 读取字典数据
    NSDictionary *dict = [NSDictionary dictionaryWithContentsOfFile:filePath];
    // 给控件赋值
    NSString *userName = dict[@"user"];
    NSString *pwd  = dict[@"pwd"];
    BOOL rmb =  dict[@"rmb"];
    BOOL autoLogin = dict[@"autoLogin"];

    _userField.text = userName;

    if (rmb == YES)
    {
        _pwdField.text = pwd;
    }

    _rmbPwdSwitch.on = rmb;
    _autoLoginSwitch.on = autoLogin;
```

- 这里目前还有问题，那就是读取出来的bool变量全是YES。等明天研究一下再。

- 问题出来了，读取字典中得bool变量时要转换一下

```objc
    // 给控件赋值
    NSString *userName = dict[@"user"];
    NSString *pwd  = dict[@"pwd"];
    BOOL rmbPwd =  [dict[@"rmb"] boolValue]; // 要转换一下，不然读取结果不准确
    BOOL login = [dict[@"autoLogin"] boolValue];
```

## 2、Preference 偏好设置

- 内部实现也是一个plist文件。不过是否系统控制路径和名称。只需写入和读取就行，不用关系细节。

- 写入数据

```objc
    // preference 偏好设置
     NSUserDefaults *userDefault = [NSUserDefaults standardUserDefaults]; // 单例对象
    [userDefault setObject:_userField.text forKey:@"user"];
    [userDefault setObject:_pwdField.text forKey:@"pwd"];
    [userDefault setBool:_rmbPwdSwitch.on forKey:@"rmb"];
    [userDefault setBool:_autoLoginSwitch.on forKey:@"autoLogin"];
```

- 读取数据

```objc
    // 偏好设置，读取数据，
    NSUserDefaults *userDefault = [NSUserDefaults standardUserDefaults];
    NSString *name = [userDefault stringForKey:@"user"];
    NSString *pwd = [userDefault stringForKey:@"pwd"];
    BOOL rmbPwd = [[userDefault boolForKey:@“rmb”] boolValue];
    BOOL login = [[userDefault boolForKey:@“autoLogin”] boolValue];
    // 给控件赋值
    _userField.text = name;
    if (rmbPwd == YES)
    {
        _pwdField.text = pwd; // 记住密码才赋值
    }
    _rmbPwdSwitch.on = rmbPwd;
    _autoLoginSwitch.on = login;
    if (login) { // 自动登录

        [self login:nil];
    }
```
- 注意：UserDefaults设置数据时，不是立即写入，而是根据时间戳定时地把缓存中的数据写入本地磁盘。所以调用了set方法之后数据有可能还没有写入磁盘应用程序就终止了。出现以上问题，可以通过调用synchornize方法强制写入
`[defaults synchornize];`

## 3、NSKeyedArchiver 归档

- 如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，可以直接用NSKeyedArchiver进行归档和恢复
- 不是所有的对象都可以直接用这种方法进行归档，只有遵守了NSCoding协议的对象才可以
- NSCoding协议有2个方法：
    - `encodeWithCoder:`
    - 每次归档对象时，都会调用这个方法。一般在这个方法里面指定如何归档对象中的每个实例变量，可以使用encodeObject:forKey:方法归档实例变量
    - `initWithCoder:`
    - 每次从文件中恢复(解码)对象时，都会调用这个方法。一般在这个方法里面指定如何解码文件中的数据为对象的实例变量，可以使用decodeObject:forKey方法解码实例变量。

- 1、常见类型

- 写入常用类型

```objc
    // 归档对象NSKeyedArchiver
    // 获取路径
    NSString *cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
    // 取得文件的全路径
    NSString *filePath = [cachePath stringByAppendingPathComponent:@"info.data"];


//            [NSKeyedArchiver archiveRootObject:_userField.text toFile:filePath];
    NSArray *all = @[_userField.text,_pwdField.text,@(_rmbPwdSwitch.on),@(_autoLoginSwitch.on)];
    [NSKeyedArchiver archiveRootObject:all toFile:filePath];
```

- 读取

```objc
    // 获取路径
    NSString *cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
    // 取得文件的全路径
    NSString *filePath = [cachePath stringByAppendingPathComponent:@"info.data"];
    // 归档对象NSKeyedArchiver
    //    NSString *name =  [NSKeyedUnarchiver unarchiveObjectWithFile:filePath];
    NSArray *all = [NSKeyedUnarchiver unarchiveObjectWithFile:filePath];
    NSLog(@"%@",all);
```

- 2、自定义对象

- 要读写自定义对象，自定义对象要遵守NSCoding协议，并实现两个方法，一个打包initWithCoder：，一个解包encodeWithCoder：。

- 以联系人为例
```objc
    #import <Foundation/Foundation.h>

    @interface SLQContact : NSObject <NSCoding>

    /*姓名*/
    @property (strong, nonatomic) NSString *name;
    // 手机号
    @property (strong, nonatomic) NSString *phone;

    + (instancetype)contactWithName:(NSString *)name andPhone:(NSString *)phone;
    @end
```

- 实现协议方法

```objc
    #import "SLQContact.h"

    @implementation SLQContact

    + (instancetype)contactWithName:(NSString *)name andPhone:(NSString *)phone
    {
        SLQContact *contact = [[self alloc] init];
        contact.name = name;
        contact.phone = phone;
        return  contact;
    }
    // 解包，按照key找属性
    - (void)encodeWithCoder:(NSCoder *)aCoder
    {
        [aCoder encodeObject:self.name forKey:@"name"];
        [aCoder encodeObject:self.phone forKey:@"phone"];
    }
    // 打包，就是给属性指定一个key
    - (id)initWithCoder:(NSCoder *)aDecoder
    {
        if (self = [super init])
        {
            self.name = [aDecoder decodeObjectForKey:@"name"];
            self.phone = [aDecoder decodeObjectForKey:@"phone"];
        }
        return  self;
    }
    @end
```

- 然后在联系人控制器中读写归档文件

- 定义一个宏用来获取文件路径

```objc
    // 获得归档文件的路径
    #define SLQFilePath [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0] stringByAppendingPathComponent:@"info.data"]
```

- 写如归档文件

```objc
    // 添加按钮，按下后进入添加联系人界面
    - (IBAction)addBtn:(id)sender {
        //
    //    [self performSegueWithIdentifier:@"contactToAdd" sender:nil];
        // 通过代码获取storyboard中得控制器
        UIStoryboard *story = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
        // 获取main。storyboard中得控制器，以标识符区分
        SLQAddViewController* addVC = [story instantiateViewControllerWithIdentifier:@"add"];
        // block封装待执行的代码
        addVC.block= ^(SLQContact *contact){
            // 更新模型
            [self.contacts addObject:contact];
            // 刷新表格
            [self.tableView reloadData];
            // 3.保存联系人,注意：如果归档数组，底层会遍历数组元素一个一个归档
            [NSKeyedArchiverarchiveRootObject:self.contactstoFile:SLQFilePath];
        };
        // 跳转到添加联系人界面
        [self.navigationController pushViewController:addVC animated:YES];
    }
```

- 读取归档文件，在属性contacts的setter方法中进行读取。

```objc
    // 重写set方法
    - (NSMutableArray *)contacts
    {
        if (_contacts == nil)
        {
            // 读取归档文件
            _contacts = [NSKeyedUnarchiver unarchiveObjectWithFile:SLQFilePath];
            if (_contacts == nil) { // 如果读取的文件为空，初始化数组
                  _contacts = [NSMutableArray array];
            }
        }
        return _contacts;
    }
```

- 还有两种存储方式，属于数据库范畴，继续学习。





http://www.cnblogs.com/songliquan/
