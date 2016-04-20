# Swift 和 OC混合开发
- 不管是OC调用swift还是swift调用oc，都需要一个桥接头文件，把要调用的那个头文件import到这个文件里，然后使用的话就按照各自的语法使用就行。

今天朋友推荐了一个有意思的页面，域名是这样的：

http://fuckingclosuresyntax.com
在 Swift 中，所有的函数都是闭包，标准的函数只不过是有名字带参数的最完整的闭包。

好了，以下是正文

 

 
## 闭包使用
- 作为变量:
  - `var 闭包名称: (参数类型) -> (返回类型)`
- 作为可选的变量:
  - `var closureName: ((parameterTypes) -> (returnType))?`
- 作为类型别名:
  - `typealias closureType = (parameterTypes) -> (returnType)`
- 作为常量:
  - `let closureName: closureType = { ... }`
- 作为调用函数时候的参数:
  - `func({(parameterTypes) -> (returnType) in statements})`
- 作为函数的参数:
  - `array.sort({ (item1: Int, item2: Int) -> Bool in return item1 < item2 })`
- 作为函数的参数并使用类型推断:
  - `array.sort({ (item1, item2) -> Bool in return item1 < item2 })`
- 作为函数的参数并推断返回类型:
  - `array.sort({ (item1, item2) in return item1 < item2 })`
- 作为函数的最后一个参数:
  - `array.sort { (item1, item2) in return item1 < item2 }`
- 作为函数的最后一个参数并且缩写参数名:
  - `array.sort { return $0 < $1 }`
- 作而函数的最后一个参数并且推断返回值:
  - `array.sort { $0 < $1 }`
- 作为函数的最后一个参数,作为一个存在函数的引用:
  - `array.sort(<)`
- 作为函数参数带默认捕获:
  - `array.sort({ [unowned self] (item1: Int, item2: Int) -> Bool in return item1 < item2 })`
- 作为函数参数带默认捕获而且推断参数类型和返回值类型:
  - `array.sort({ [unowned self] in return item1 < item2 })`