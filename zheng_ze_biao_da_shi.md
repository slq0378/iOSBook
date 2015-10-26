
# 正则表达式

- 正则表达式是对字符串操作的一种逻辑公式，用事先定义好的一些特定字符、及这些特定字符的组合，组成一个"规则字符串"，这个"规则字符串"用来表达对字符串的一种过滤逻辑。

## 正则表达式的用处:

- 1. 判断给定的字符串是否符合某一种规则(专门用于操作字符串)
   	- 电话号码，电子邮箱，URL...
   	- 可以直接百度别人写好的正则
   	- 别人真的写好了，而且测试过了，我们可以直接用
   	- 要写出没有漏洞正则判断，需要大量的测试，通常最终结果非常负责
- 2. 过滤筛选字符串，网络爬虫
- 3. 替换文字，QQ聊天，图文混排

## 语法规则

- 1>	集合

```
	[xyz]		字符集合(x/y或z)
	[a-z]		字符范围
	[a-zA-Z]
	[^xyz]		负值字符集合 (任何字符, 除了xyz)
	[^a-z]		负值字符范围
	[a-d][m-p]  并集(a到d 或 m到p)
```
- 2>	常用元字符

```
	.			匹配除换行符以外的任意字符
	\w			匹配字母或数字或下划线或汉字 [a-zA-Z_0-9]
	\s			匹配任意的空白符（空格、TAB\t、回车\r \n）
	\d			匹配数字 [0-9]
	^			匹配字符串的开始
	$			匹配字符串的结束
	\b			匹配单词的开始或结束
```
- 2>	常用反义符

```
	\W          匹配任意不是字母，数字，下划线，汉字的字符[^\w]
	\S			匹配任意不是空白符的字符 [^\s]
	\D			匹配任意非数字的字符[^0-9]
	\B			匹配不是单词开头或结束的位置
	[^x]		匹配除了x以外的任意字符
	[^aeiou]	匹配除了aeiou这几个字母以外的任意字符
```

- 4>	常用限定符

```
	*			重复零次或更多次
	+			重复一次或更多次
	?			重复零次或一次
	{n}			重复n次
	{n,}		重复n次或更多次
	{n,m}		重复n到m次,
```

- 5>	贪婪和懒惰

```
	*?			重复任意次，但尽可能少重复
	*+			重复1次或更多次，但尽可能少重复
	??			重复0次或1次，但尽可能少重复
	{n,m}?      重复n到m次，但尽可能少重复
	{n,}?		重复n次以上，但尽可能少重复
```


## 使用过程
- 1、创建规则
- 2、创建正则表达式对象
- 3、开始匹配

## 代码示例

```swift
    private func check(str: String) {
        // 使用正则表达式一定要加try语句
        do {
            // - 1、创建规则
            let pattern = "[1-9][0-9]{4,14}"
            // - 2、创建正则表达式对象
            let regex = try NSRegularExpression(pattern: pattern, options: NSRegularExpressionOptions.CaseInsensitive)
            // - 3、开始匹配
            let res = regex.matchesInString(str, options: NSMatchingOptions(rawValue: 0), range: NSMakeRange(0, str.characters.count))
            // 输出结果
            for checkingRes in res {
                print((str as NSString).substringWithRange(checkingRes.range))
            }
        }
        catch {
            print(error)
        }
    }
```

- 其他几个常用方法

```
            // 匹配字符串中所有的符合规则的字符串, 返回匹配到的NSTextCheckingResult数组
            public func matchesInString(string: String, options: NSMatchingOptions, range: NSRange) -> [NSTextCheckingResult]

            // 按照规则匹配字符串, 返回匹配到的个数
            public func numberOfMatchesInString(string: String, options: NSMatchingOptions, range: NSRange) -> Int

            // 按照规则匹配字符串, 返回第一个匹配到的字符串的NSTextCheckingResult
            public func firstMatchInString(string: String, options: NSMatchingOptions, range: NSRange) -> NSTextCheckingResult?

            // 按照规则匹配字符串, 返回第一个匹配到的字符串的范围
            public func rangeOfFirstMatchInString(string: String, options: NSMatchingOptions, range: NSRange) -> NSRange
```

## 使用子类来匹配日期、地址、和URL

- 看官网文档解释，可以知道这个`NSDataDetector`主要用来匹配日期、地址、和URL。在使用时指定要匹配的类型


```swift
public class NSDataDetector : NSRegularExpression {
    // all instance variables are private

    /* NSDataDetector is a specialized subclass of NSRegularExpression.  Instead of finding matches to regular expression patterns, it matches items identified by Data Detectors, such as dates, addresses, and URLs.  The checkingTypes argument should contain one or more of the types NSTextCheckingTypeDate, NSTextCheckingTypeAddress, NSTextCheckingTypeLink, NSTextCheckingTypePhoneNumber, and NSTextCheckingTypeTransitInformation.  The NSTextCheckingResult instances returned will be of the appropriate types from that list.
    */

    public init(types checkingTypes: NSTextCheckingTypes) throws

    public var checkingTypes: NSTextCheckingTypes { get }
}

// 这个是类型选择
    public static var Date: NSTextCheckingType { get } // date/time detection
    public static var Address: NSTextCheckingType { get } // address detection
    public static var Link: NSTextCheckingType { get } // link detection
```

-  `NSDataDetector` 获取URL示例

```swift
    /**
    匹配字符串中的URLS
    - parameter str: 要匹配的字符串
    */
    private func getUrl(str:String) {
        // 创建一个正则表达式对象
        do {
            let dataDetector = try NSDataDetector(types: NSTextCheckingTypes(NSTextCheckingType.Link.rawValue))
            // 匹配字符串，返回结果集
            let res = dataDetector.matchesInString(str, options: NSMatchingOptions(rawValue: 0), range: NSMakeRange(0, str.characters.count))
            // 取出结果
            for checkingRes in res {
                print((str as NSString).substringWithRange(checkingRes.range))
            }
        }
        catch {
            print(error)
        }
    }
```

- ".*?" 可以满足一些基本的匹配要求
- 如果想同时匹配多个规则 ，可以通过 "|" 将多个规则连接起来
- 将字符串中文字替换为表情

```swift
    /**
    显示字符中的表情
    - parameter str: 匹配字符串
    */
    private func getEmoji(str:String) {
        let strM = NSMutableAttributedString(string: str)
        do {
            let pattern = "\\[.*?\\]"
            let regex = try NSRegularExpression(pattern: pattern, options: NSRegularExpressionOptions.CaseInsensitive)
            let res = regex.matchesInString(str, options: NSMatchingOptions(rawValue: 0), range: NSMakeRange(0, str.characters.count))
            var count = res.count
            // 反向取出文字表情
            while count > 0 {
                let checkingRes = res[--count]
                let tempStr = (str as NSString).substringWithRange(checkingRes.range)
                // 转换字符串到表情
                if let emoticon  = EmoticonPackage.emoticonWithStr(tempStr) {
                    print(emoticon.chs)
                    let attrStr = EmoticonTextAttachment.imageText(emoticon, font: 18)
                    strM.replaceCharactersInRange(checkingRes.range, withAttributedString: attrStr)
                }
            }
            print(strM)
            // 替换字符串,显示到label
            emoticonLabel.attributedText = strM
        }
        catch {
            print(error)
        }
    }
```

## `TextKit` 给URL高亮显示

- 主要用到三个类
- `NSTextStorage`
- `NSLayoutManager`
- `NSTextContainer`

### 自定义UILabel来实现url高亮

- 1、定义要用到的属性

```swift
  /*
    只要textStorage中的内容发生变化, 就可以通知layoutManager重新布局
    layoutManager重新布局需要知道绘制到什么地方, 所以layoutManager就会文textContainer绘制的区域
    */

    // 准们用于存储内容的
    // textStorage 中有 layoutManager
    private lazy var textStorage = NSTextStorage()

    // 专门用于管理布局
    // layoutManager 中有 textContainer
    private lazy var layoutManager = NSLayoutManager()

    // 专门用于指定绘制的区域
    private lazy var textContainer = NSTextContainer()


    override init(frame: CGRect) {
        super.init(frame: frame)
        setupSystem()
    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        setupSystem()
    }

    private func setupSystem()
    {
        // 1.将layoutManager添加到textStorage
        textStorage.addLayoutManager(layoutManager)
        // 2.将textContainer添加到layoutManager
        layoutManager.addTextContainer(textContainer)
    }
    override func layoutSubviews() {
   		 super.layoutSubviews()
    	 // 3.指定区域
   		 textContainer.size = bounds.size
    }
```

- 2、重写label的text属性

```swift
     override var text: String?
        {
        didSet{
            // 1.修改textStorage存储的内容
            textStorage.setAttributedString(NSAttributedString(string: text!))

            // 2.设置textStorage的属性
            textStorage.addAttribute(NSFontAttributeName, value: UIFont.systemFontOfSize(20), range: NSMakeRange(0, text!.characters.count))

            // 3.处理URL
            self.URLRegex()

            // 2.通知layoutManager重新布局
            setNeedsDisplay()
        }
    }
```

- 3、匹配字符串

```swift
    func URLRegex()
    {
        // 1.创建一个正则表达式对象
        do{
            let dataDetector = try NSDataDetector(types: NSTextCheckingTypes(NSTextCheckingType.Link.rawValue))

            let res =  dataDetector.matchesInString(textStorage.string, options: NSMatchingOptions(rawValue: 0), range: NSMakeRange(0, textStorage.string.characters.count))

            // 4取出结果
            for checkingRes in res
            {
                let str = (textStorage.string as NSString).substringWithRange(checkingRes.range)
                let tempStr = NSMutableAttributedString(string: str)

//                tempStr.addAttribute(NSForegroundColorAttributeName, value: UIColor.redColor(), range: NSMakeRange(0, str.characters.count))
                tempStr.addAttributes([NSFontAttributeName: UIFont.systemFontOfSize(20), NSForegroundColorAttributeName: UIColor.redColor()], range: NSMakeRange(0, str.characters.count))

                textStorage.replaceCharactersInRange(checkingRes.range, withAttributedString: tempStr)
            }
        }catch
        {
            print(error)
        }

    }
```

- 4、重绘文字

```swift
    // 如果是UILabel调用setNeedsDisplay方法, 系统会促发drawTextInRect
    override func drawTextInRect(rect: CGRect) {
        // 重绘
        // 字形 : 理解为一个小的UIView
        /*
        第一个参数: 指定绘制的范围
        第二个参数: 指定从什么位置开始绘制
        */
        layoutManager.drawGlyphsForGlyphRange(NSMakeRange(0, text!.characters.count), atPoint: CGPointZero)
    }
```

### 获取label中URL的点击
- 如果要获取URL的点击，那么必须获取点击的范围

```swift
    override func touchesBegan(touches: Set<UITouch>, withEvent event: UIEvent?) {
        // 1、获取手指点击的位置
        let touch = (touches as NSSet).anyObject()!
        let point = touch.locationInView(touch.view)
        print(point)
        // 2、获取URL区域
        // 注意: 没有办法直接设置UITextRange的范围
        let range = NSMakeRange(10, 20)
        // 只要设置selectedRange, 那么就相当于设置了selectedTextRange
        selectedRange = range

        // 给定指定的range, 返回range对应的字符串的rect
        // 返回数组的原因是因为文字可能换行
        let array = selectionRectsForRange(selectedTextRange!)

        for selectionRect in array {
            if CGRectContainsPoint(selectionRect.rect, point) {
                print("点击了URL")
            }
        }
   }
```
