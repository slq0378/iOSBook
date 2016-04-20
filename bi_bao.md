# 闭包
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


### 传递点击事件
- 在子控件中封装点击操作，通过闭包传递事件
  - 首先声明一个闭包属性 ` var buttonClicked:(()->())?`
  - 其次是给闭包赋值，在初始化方法中使用尾随闭包传递
  - 最后就是在触发事件是调用闭包


```swift
class SLQPandaCellView: UIView {
    // 1、声明一个闭包属性，类似Block属性
    var buttonClicked:(()->())?
    
    /**
     初始化方法
     
     - parameter title:   图片
     - parameter name:    文字
     - parameter clicked: 尾随闭包，可以在函数后加{}调用
     
     - returns: 当前对象
     */
    init(title:String , name:String,clicked: (()->())) {
        super.init(frame:CGRectZero)
        // 2、给闭包赋值
        buttonClicked = clicked
        // button
        backgroundColor = UIColor.whiteColor()
        let button = UIButton()
        addSubview(button)
        button.snp_makeConstraints { (make) in
            make.top.equalTo(self).offset(10)
            make.width.equalTo(kButtonWidth-20)
            make.height.equalTo(self).offset(-20)
            make.left.equalTo((ScreenWidth - kLabelWidth - kButtonWidth-10)/2)
        }
        button.setBackgroundImage(UIImage.init(named: title), forState: UIControlState.Normal)
        
        // label
        let label = UILabel()
        self.addSubview(label)
        label.snp_makeConstraints { (make) in
            make.top.equalTo(self)
            make.width.equalTo(kLabelWidth)
            make.height.equalTo(self)
            make.left.equalTo(button.snp_right).offset(10)
        }
        label.textAlignment = NSTextAlignment.Center
        label.textColor = UIColor.darkTextColor()
        label.text = name
        
        let bg = UIView.init()
        addSubview(bg)
        bg.snp_makeConstraints { (make) in
             make.edges.equalTo(EdgeInsets(top: 0, left: 0, bottom: 0, right: 0))
        }
        let tap = UITapGestureRecognizer.init(target: self, action: #selector(SLQPandaCellView.tapGes));
        bg.addGestureRecognizer(tap)
        
    }
    
    
    func tapGes() -> Void{
        // 3、这里调用闭包，回调事件
        buttonClicked!()
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
}
```



