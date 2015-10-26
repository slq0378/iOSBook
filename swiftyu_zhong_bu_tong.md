# Swift 与众不同的地方

## switch（元组）
- 特点
	- 其他语言中的switch语句只能比较离散的整形数据（字符可以转换成整数）
	- 但是swift中可以比较整数、浮点数、字符、字符串、和元组数据类型，而且它可以是离散的也可以使连续的范围
	- 而且在swift中case语句不需要显示的添加break语句，分支语句会自动进行跳转
	- 每个switch语句至少有一个default语句

- switch中比较元组
	- 可以在元组中进行值绑定，来对某一个值进行判断
	- 可以使用where 语句对元组中的条件进行判断

## 跳转语句
- break、contune、fallthrough、return

### break
- 主要用户循环体中终止循环
- break的使用有两种方式

	```swift
	break	// 	没有标签
	break label // 添加标签
	// break语句中标签的使用
	label1: for var x = 0; x < 5; x++ { // 标签1
	    label2: for var y = 0; y > 0; y-- { // 标签2
	        if x == y {
	        break label1 // 默认是跳出标签2，加上标签，可以直接跳出到指定循环
	        }
	        print("(x,y) = (\(x),\(y))")
	    }
	}
	```

### continue

- `continue` 的使用和break类似，可以使用标签指定继续的循环

	```objc
	// continue语句中标签的使用
	label1: for var x = 0; x < 5; x++ { // 标签1
	    label2: for var y = 0; y > 0; y-- { // 标签2
	        if x == y {
	            continue label1 // 默认是跳出标签2，加上标签，可以直接跳出到指定循环
	        }
	        print("(x,y) = (\(x),\(y))")
	    }
	}
	```

### fallthrough
- fallthrough是贯通语句，在swift的swiftch语句中case语句不能贯通，但是如果我们需要设置贯通语句，可以使用fallthrough语句
- 在switch语句中case后fallthrough，会执行下一个case语句

### return
- 函数返回值

## 数组和字典
- 数组
	- 和oc中数组没啥大的区别
- 字典
	- 字典的遍历有两种方式，一种是遍历keys，一种是遍历values，或者同时遍历
- 集合的复制
	- 值传递和引用传递

## 函数
- 参数传递
	- 外部参数名和内部参数名
	- 外部参数名可以在调用函数时显示出来

	```objc
	func rectangle(W width:Int, H height:Int) -> Int {
		// 	其中W\H是外部参数名，在外部调用这个方法时会显示出来，提示这个参数是神马意思
		// width\height 是内部参数，在函数内部调用使用
		return width * height
	}
	```

	- 也可以让内部便令名变为外部变量名，使用字符 “#”

	```objc
	func rectangle(#width:Int, #height:Int) -> Int{
		// 	其中W\H是外部参数名，在外部调用这个方法时会显示出来，提示这个参数是神马意思
		// width\height 是内部参数，在函数内部调用使用
		return width * height
	}
	```

- 参数默认值

	```objc
	func rectangle(#width:Int = 100, #height:Int = 100) -> Int {
		// 	其中W\H是外部参数名，在外部调用这个方法时会显示出来，提示这个参数是神马意思
		// width\height 是内部参数，在函数内部调用使用
		return width * height
	}
	```

- 可变参数

	```objc
	func sum(numbers: Double... ) -> Double {
		var total: Double = 0
		for num in numbers {
			total += num
		}
		return total
	}
	```

- 参数的引用传递
	- 在众多数据类型中，只有类是引用传递，其他的数据类型如整形、浮点型、布尔型、字符串、元组、集合、数据、枚举、结构体都是值传递。
	- 如果一定要把一个值传递的类型的数据变为引用传递模型的话，可使用关键字'inout'

	```objc
	func rectangle(inout width:Int, increased:Int = 1 ) {
		width += increased
	}
	var value:Int = 10
	rectangle(&value) // 结果是11
	rectangle(&value,100) // 结果是111
	```
- 函数返回值
	- 特别之处是可以返回一个元组，表示多个值

	 ```objc
	func rectangleArea(width:Double, increased:Double = 1 ) -> (area:Double,height:Double) //返回面积和高度
	```

- 函数类型
	- 函数类型包括函数参数类型和返回值类型
	- `(Double, Double） -> Double //传入宽高，计算面积`
	- 这个就是函数类型，函数类型可以作为其他函数的返回类型
	- 比如写一个通用的方法来计算更多种图形的面积，这时可以使用这个函数类型作为返回值
	- 函数类型还可以作为参数类型使用，可以直接使用返回值作为参数，然后可以在函数内部调用这个函数
- 函数重载
	- 和C++中函数重载类似，但是在swift中`函数返回值类型`、`外部参数名` 也可以作为不同函数的判断标准
- 嵌套函数
	- 可以在函数内部定义并调用函数

## 泛型和泛型函数
- 泛型就是在运行时才确定类型的一种机制,这个C++中的泛型如出一辙

	```objc
		func add(a:Int, b:Int) -> Int
		{
			return a + b
		}
		func add(a:Double, b:Double) -> Double
		{
			return a + b
		}
	```

- 如果用泛型计算两个数的和，可以写成下面这个样

	```objc
		func add<T>)(a:T, b:T) -> T {
			return a + b
		}
		// 或者指定多重类型
		func add<T,U>(a:T, b:U) -> T {
			return a + T(b)
		}
	```

- 如果比较两个类型的话，那么必须要遵守`Comparable`协议才可以

	```objc
		func isEquals<T: Comparable>(a:T, b:T) -> Bool {
			return (a == b)
		}
	```

## 闭包
- 闭包是自包含的匿名函数代码块，可以作为表达式、函数参数和函数返回值。

	```swift
	{ (参数列表) -> 返回值类型 in
			语句组
	}
	```

### 使用举例
- 完整版本

	```swift
	{(a:Int, b:Int) -> Int in
			return a + b
		}
	```

- 简化版本 ` {a,b in return a + b }`
	- 如果闭包中只有一条return语句，那么return 语句可以省略` {a,b in a + b }`
	- 缩写参数名称 `{$0 + $1}` swift会自动推到出参数类型，$0表示第一个参数，$1表示第二个参数
	- 闭包作为返回值,直接可以为变量和常量赋值

	```swift
	 let reslut:Int = {(a:Int, b:Int) -> Int in
			return a + b
		}(10, 89)
	```

- 捕获上下文中的变量和常量
	- 嵌套函数和闭包可以访问其所在上下文中的常量和变量，这就是`捕获值`。即便是定义这些常量或变量的作用域已经不存在，嵌套函数或闭包仍然可以在函数体内或闭包内修改或引用这些值。

## swift中的面向对象

### 枚举
- swift中枚举可以存储多种数据类型，并不是真正的整数

	``` swift
		// 默认并不是整形
		enum WeekDays {
			case Monday
			case Tuesday
			case Wednesday
			case Thursday
			case Friday
		}
		// 简写
			enum WeekDays {
			case Monday, Tuesday, Wednesday, Thursday, Friday
		}
	```

- 注意：在switch中使用枚举类型时，语句中的case必须全部包含枚举中所有成员，不能多也不能少，包括default语句。
- 指定没枚举的原始值，可以是字符、字符串、整数、浮点数等


	``` swift
		// 指定枚举的原始值
		enum WeekDays: Int {
			case Monday = 0
			case Tuesday
			case Wednesday
			case Thursday
			case Friday
		}
	```

- 原始值使用的话，要用到函数 `toRaw()` 和 `fromRaw()`

### 结构体
- swift中结构体和类非常类似，可以定义成员变量和方法
- 结构体不具备继承、运行时强制类型转换、析构函数、引用计数等
- 除了对象是引用传递，其他数据类型都是值传递，引用之间的比较可以使用 ‘===’ 和 ‘!===’

## 可选类型和可选链
- “?” 和 "!"
	- 不确定一个变量是否有值，可以加 ?
- 可选绑定 : 如果赋值不为nil的话就执行if语句

	```swift
	if let result: Double? = divide(100,0) {
		print("success")
	}
	else {
		print("failure")
	}
	```

- 强制拆封 : 使用 ！ 来强制拆封

	```swift
	let result: Double? = divide(100,0)
		print(result!)
	```

- 可选链
	- 一些可选类型内部包含其他可选类型，然后产生层级关系。

## 访问限定
- 访问范围的界定主要有：模块和源文件
- 访问级别：public、internal、private
- 可修饰类、结构体、枚举等面向对象的类型，还可以修饰变量、常量、下标、元组、函数、属性等。以上这些统称“实体”
- 使用方式
	 - `public` : 只要在import进所在模块，那么在该模块中都可访问
	 - `internal` : 只能访问自己模块的任何internal实体，不能访问其他模块中的internal实体，默认就是internal修饰符
	 - `private` : 只能在当前源文件中访问的实体。
- 设计原则
	- 如果是自己在本文件内部使用，就用默认的就行。
	- 如果是开发第三方框架的话，那么一定要设计好访问级别，接口一定要public。其他的可以private

## 属性和下标

### 1、存储属性
- 存储属性适用于类和结构体两种类型，包括常量属性（`let`）和变量属性（`var`）
	- 在一个类中定义一个结构体对象属性或者类对象属性时就算是存储属性，可以指定默认值
- 可以对存储属性进行延迟加载
- 属性观察者
	- 使用在一般的存储属性和计算属性上
	- `willSet` 在设置新值之前调用
	- `didSet` 在设置新值之后马上调用

	```swift
 	var fullName: String {
        didSet {
             fullName = firstName + "." + secondName
        }
    ```

### 2、计算属性
- 计算属性本身不存储数据，只从其他存储属性中获得数据。类、结构体、枚举都可以定义计算属性
- 计算属性只能使用`var`修饰

	```swift
    var firstName: String = "Jone"
    var secondName: String = "Hu"
    // 计算属性
    var fullName: String {
        get {
            return firstName + "." + secondName
        }
        set (newValue) {
            let names = newValue.componentsSeparatedByString(".")
            firstName = names[0]
            secondName = names[1]
        }
    }
	```

- 只读计算属性，只写set方法即可，这是可以省略`set`，直接`return`即可

	```swift
    var firstName: String = "Jone"
    var secondName: String = "Hu"
    // 计算属性
    var fullName: String {
       return firstName + "." + secondName
    }
	```

### 3、静态属性
- 可以在类、结构体、枚举中定义静态属性

### 4、下标
- 一些集合类型，可以使用下标访问


	```swift
	/**
	*  定义二维数组，使用下标访问数组
	*  内部还是一个一维数组，不过展示给外界的时一个二维访问方式
	*/
	struct DoubleArray {
	    // 属性定义
	    let rows: Int
	    let cols: Int
	    var grid: [Int]

	    // 构造方法
	    init(rows: Int, cols: Int) {
	        self.rows = rows
	        self.cols = cols
	        grid = Array(count: rows * cols, repeatedValue: 0) // 利用泛型创建一个数组
	    }

	    // 下标定义
	    subscript(row: Int, col: Int) -> Int {
	        get {
	            return grid[row * col + col]
	        }
	        set (newValue) {
	            grid[row * col + col] = newValue
	        }
	    }
	}

	// 使用二维数组
	var arr2 = DoubleArray(rows: 10, cols: 10)

	// 初始化二维数组
	for var i = 0; i < 10; i++ {
	    for var j = 0; j < 10; j++ {
	        arr2[i,j] = i * j
	    }
	}

	// 输出二维数组
	for var i = 0; i < 10; i++ {
	    for var j = 0; j < 10; j++ {
	        print("\t \(arr2[i,j])")
	    }
	    print("\n")
	}
	```

## 方法
- 类、结构体、枚举中都可以定义方法
- 这里主要说结构体和枚举中方法的定义
	- 默认情况下，结构体和枚举中的方法是不能修改属性的
	- 如果要修改属性的话，需要在方法之前加关键字 `mutating` ,称之为**变异方法**

### 静态方法
- 结构体和枚举中静态方法用`static`，类中静态方法使用`class`
- 1、结构体中的静态方法
	- 不能访问示例属性和示例方法

	```swift
	/**
	*  结构体中的静态方法
	*/
	struct Account {
	    var owner: String = "Jack"
	    static var interestRate: Double = 0.668
	    // 静态方法
	    static func interestRateBy(amount: Double) -> Double  {
	        return interestRate * amount
	    }
	    func messageWith(amount: Double) -> String {
	        let interest  = Account.interestRateBy(amount)
	        return "\(self.owner) 的利息是 \(interest)"
	    }
	}
	// 调用静态方法
	print(Account.interestRateBy(10_000.00))
	//
	var myAccount = Account()
	print(myAccount.messageWith(10_000))
	```

- 2、枚举中的静态方法
	- 不能访问示例属性和示例方法

	```swift
	/// 枚举中的静态方法
	enum Account1 {
	    case 中国银行
	    case 中国工商银行
	    case 中国建设银行
	    case 中国农业因银行
	    static var interestRate: Double = 0.669
	    // 静态方法定义
	    static func interestBy(amount: Double) -> Double {
	        return interestRate * amount
	    }
	}
	// 静态方法的调用
	print(Account1.interestBy(10_000.00))
	```

- 3、类中的静态方法
	- 不能访问示例属性和示例方法

	```swift
	/// 类中的静态方法，使用关键字class
	class Account2 {
	    var ower: String = "Jack"
	    class func interestBy(amount: Double) -> Double {
	        return 0.889 * amount
	    }
	}
	// 调用静态方法
	print(Account2.interestBy(10_000.00))
	```

## 构造和析构
- 只有类和结构体才有构造`init()`和析构函数`deinit`
- 在创建实例过程中需要一些初始化操作，这个过程称为*构造*
- 在实例最后被释放的过程中，需要清楚资源，这个过程称为*析构*
- *注意： 构造器 可以重载，没有返回值*

### 构造器
- 构造器主做一些属性的初始化操作
- 如果不写init方法，那么会自动生成一个空的init构造器
- 如果有继承关系，要先调用父类的构造器
- 横向代理与向上代理
	- 横向代理：构造器调用发生在本类内部，添加`convenience`关键字即可作为便利构造器，便利构造器必须调用本类内部的其他构造器
	- 向上代理：有继承关系，就先调用父类的构造器
### 析构器
- 析构器只作用于类
- 析构器没有返回值，没有参数，不能重载

## 继承
- swift单继承
- 重写属性、下标以及方法
- 重写父类的方法使用`override`
- 终止属性或者方法的继承可以使用 `final`
- 类型检查 `is` `as` `Any` `AnyObject`
	- `is` 类型判断
	- `as` 类型转换
	- `Any` 任何类型，包括类和其他数据类型
	- `AnyObject` 任何类
## 扩展和协议

### 扩展`extension`
- 扩展的类型可以使类、结构体、枚举
- 扩展可以增加属性和方法
- 扩展计算属性、方法、构造器、下标
- **注意：扩展类的构造器的时只能增加遍历构造器，指定构造器和析构器只能由源类型指定**

### 协议 `protocol`
- 类似C++中的纯虚类
- 可以定义属性和方法
- 协议可以继承协议

## swift内存管理
- swift内存管理遵循ARC特性
- 对引用类型进行引用计数，基本数据类型由系统管理
### 循环引用问题
- swift中解决循环引用由两种方式
	- 1、弱引用（`weak reference`）
	- 2、无主引用(`unowned reference`)
- 弱引用可以没有值，因此必须设置成可选类型
- 无主引用适用于引用对象永远有值

### 闭包中的循环引用问题
- 闭包中的循环引用的解决方法和上面的一样，使用弱引用和无主引用

	```swift
	class Person {
	    let firstName: String = "Jack"
	    let secondName: String = "lu"
	    /**
	    *  解决闭包强循环引用
	    */
	    lazy var fullName: (String, String) -> String = {
	        // [weak self] (firstName: String, secondName: String) -> String in
	        [unowned self] (firstName: String, secondName: String) -> String in
	        return firstName + secondName
	    }
	}
	```

## OC与Swift

- OC和swift可以混合开发

- 1、swift 调用 OC
	- 桥接文件 "<工程名>-Bridging-Header.h"
	- 在桥接文件中引入OC头文件即可
- 2、OC调用swift
	- 包含头文件，这个头文件命名样式 "<工程名>-swift.h"
	- 在swift源文件中要把类声明为 `@objc`
	- 这时新建`swift`类就不需要选择新建swift文件，而是选择`Cocoa Touch Class`,然后选择语言类型位swift

## 实战

- 1、需求分析
	- 用户群：确定大体功能
	- 确定应用发布平台
	- 原型设计：绘出草图
- 2、分层架构设计
	- 表示层->业务逻辑层->数据持久层->信息系统层
- 3、设计
	- 纯swift实现


- 闭包
- 循环引用
- `weak var weakSelf = self;`
- 在Swift中, 如果在某个类中定义一个属性, 那么这个属性必须要初始化, 否则就会报错
- 如果暂时不想初始化, 那么可以在后面写上一个?号


- 懒加载

```objc
    // 格式: 定义变量时前面使用lazy来修饰变量, 后面通过等到赋值一个闭包
    // 注意点: 1.必须是用var 2.闭包后面必须跟上()
    lazy var dataList:[String] = { () -> [String] in
        print("加载数据")
        return ["zs","ls","ww","mz"]
    }()
    // 如果闭包是用于懒加载, 那么in之前的代码都可以删除包括in在内
    lazy var dataList2:[String] = {
        print("加载数据")
        return ["zs","ls","ww","mz"]
    }()
```

- 构造方法

```objc
import UIKit
class Person: NSObject {
    // 如果定义属性的时候没有初始化, 那么必须在后面写上一个?
    // Swift要求, 属性是必须有初始值的
    // 只要在构造方法中对属性进行了初始化, 那么就不用写?
    var name:String?
    /*
    如果是定义一个 "对象属性" 那么后面可以写上 ?
    如果是定义一个 "基本数据类型属性", 那么建议直接赋值为0
    因为super.init()方法在分配存储空间的时候, 如果发现属性是一个对象, 并且是一个可选类型, 那么会给这个属性分配存储空间
    但是如果属性是一个基本数据类型, 并且是可选类型, 那么super.init()不会给该属性分配存储空间
    */
    //    var age:Int?
    var age:Int = 0
    // 注意: 如果自定义了构造方法, 并且没有重写父类默认的构造方法
    // 那么默认的构造方法就会失效
    override init() {
        self.name = "Mr Song"
        self.age = 25
    }
    // 自定义构造方法
    // Swift中有方法重载的概念
    // 允许有同名的方法, 只要形参或返回值不一样即可
    init(name:String , age:Int)
    {
        self.name = name
        self.age = age
    }
    init(dict:[String:NSObject])
    {
        super.init();
        // 注意点: Swift中如果想在构造方法中使用KVC给属性赋值
        // 那么在使用KVC之前必须调用super.init()
        // 调用super.init()的目的就是为了能在在KVC赋值之前给属性分配存储空间
        setValuesForKeysWithDictionary(dict)
    }
}
```

- setter/getter

```objc
    // setter/getter
    var _name:String?
    var name:String?{
        get{
            return _name;
        }
        set{
            // 只要外界给通过对象.name给name赋值, 那么值就会保存在newValue
            _name = newValue
        }
    }
    var sex:String?{
        // 设置完值之后调用
        // Swift中使用didSet来替代OC中重写setter方法
        didSet{
            print(self)
            print(sex)
        }
    }
    // 如果只重写了get方法, 那么这个属性我们称之为 计算型 属性
    // 也就是对应OC中的只读属性
    // 注意点: 计算型属性是不占用内存空间的
    var age:Int{
        get{
            return 30
        }
    }
```

- UITableView在swift中使用

```objc
import UIKit
// Swift中遵守协议直接在后面通过逗号分隔即可
class ViewController: UIViewController{
    override func loadView() {
        let vc = UITableView()
        vc.frame = UIScreen.mainScreen().bounds
        vc.dataSource = self
        vc.delegate = self
        view = vc
    }
    // MARK: - 懒加载
    lazy var dataList:[String] = {
        return ["ls","zs","wa"]
    }()
}
// MARK: - 分类
// 苹果官方建议, 可以将数据源代理方法单独写到一个扩展中, 以便于提高代码的可读性
// extension 相当于OC中的 catogory
extension ViewController: UITableViewDataSource, UITableViewDelegate
{
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return dataList.count
    }
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        var cell = tableView.dequeueReusableCellWithIdentifier("cell")
        if cell == nil{
            cell = UITableViewCell(style: UITableViewCellStyle.Default, reuseIdentifier: "cell")
        }
        cell?.textLabel?.text = "第\(indexPath.row)行"
        return cell!
    }
}
```


## swift中字符串转换成类
- swift中的类前会有一个命名空间，这里注意获取并添加到类前面

```objc
// 0、通过字符串创建类
// 0.1、获取命名空间
let str = NSBundle.mainBundle().infoDictionary!["CFBundleExecutable"] as! String
// 0.2、字符串转换成类
let cls:AnyClass? = NSClassFromString(str +  "." + childControllerName)
// 0.3、将类转换成具体类
let vcCls = cls as! UIViewController.Type
// 0.4、初始化类
let vc = vcCls.init()
```

## Swift中如何定义协议: 必须遵守NSObjectProtocol

```objc
// Swift中如何定义协议: 必须遵守NSObjectProtocol
protocol SLQDefaultViewDelegate : NSObjectProtocol
{
    // 登录回调
    func loginBtnWillClick()
    // 注册回调
    func registerBtnWillClick()

```

### KVC
注意:基本数据类型要由初始值
然后重写init方法,首先要调用super.init()
直接使用kvc方法`setValuesForKeysWithDictionary(dict)`
最后重写setValue:value : forUnderfinedKey key:

### 判断软件版本
- 用swift和用oc差不多，就是一些方法的使用有些具体的区别

```objc
func isNewUpdate() -> Bool{
    // 1.获取当前软件的版本号info.plist中
    let currentVersion = NSBundle.mainBundle().infoDictionary!["CFBundleShortVersionString"] as! String
    // 2.获取以前的软件版本号 --> 从本地文件中读取(以前自己存储的)
    // ？？ 表示如果前面的数据获取失败，默认使用？？后面制定的
    let sandBoxVersion = NSUserDefaults.standardUserDefaults().objectForKey("CFBundleShortVersionString") as? String ?? ""
    // 比较版本，判断是不是新的版本
    // 3.比较当前版本号和以前版本号
    if currentVersion.compare(sandBoxVersion) == NSComparisonResult.OrderedDescending {
        // 3.1如果当前>以前 --> 有新版本
        // 3.1.1存储当前最新的版本号
        // iOS7以后就不用调用同步方法了
        NSUserDefaults.standardUserDefaults().setObject(currentVersion, forKey: "CFBundleShortVersionString")
        return true
    }
    return false
}
```
### 字典转模型注意

```objc
    /// 将字典数组转换为模型数组
    class func dict2Model(list: [[String: AnyObject]]) -> [Status] {
        var models = [Status]()
        for dict in list
        {
            models.append(Status(dict: dict))
        }
        return models
    }
    // 打印当前模型
    var properties = ["created_at", "id", "text", "source", "pic_urls"]
    override var description:String {
        let dict = dictionaryWithValuesForKeys(properties)
        return "\(dict)"
    }
```


