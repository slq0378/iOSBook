# 表情键盘实现
- 直接自定义文本输入框的`inputView`属性，直接改成自己想要的UIView即可。
- 表情保存在本地，从沙盒中读取


## emoji表情显示
- emoji对应十六进制,如果要显示的话，需要转换一下


```swift
    // emoji对应十六进制
    func emojiToShow(code: String) -> String {
        // 创建扫描器
        let scanner = NSScanner(string: code)
        // 将十六进制数据转换成字符串
        var result: UInt32 = 0
        scanner.scanHexInt(&result)
        // 字符串转换成emoji字符串
        let emojiStr = Character(UnicodeScalar(result))
        return String(emojiStr)
    }
```

## 图片显示
- 在文本输入框里显示小图片作为表情使用


```swift
		 // 1、显示图片，通过富文本
		 let attachment = NSTextAttachment()
		 // 然后加载图片
		 attachment.image = UIImage(contentsOfFile: emoticon.imagePath!)
		 // 设置附件大小,稍微下移一点，图片比较答，而emoji比较小
		 attachment.bounds = CGRect(x: 0, y: -4, width: 20, height: 20)
		 // 2、根据附件创建属性字符串
		 let imageText = NSAttributedString(attachment: attachment)
		 // 3、拿到当前的所有字符串内容
		 let strM = NSMutableAttributedString(attributedString: self.textView.attributedText)
		 // 4.插入表情到当前光标所在的位置
		 let range = self.textView.selectedRange
		 strM.replaceCharactersInRange(range,withAttributedString: imageText)
		 // 设置附件尺寸
		 strM.addAttribute(NSFontAttributeName,value: UIFont.systemFontOfSize(19),range: NSMakeRange(range.location, 1))
		 // 5、重新赋值
		 self.textView.attributedText = strM
		 // 设置光标位置
		 self.textView.selectedRange = NSMakeRange(range.location  + 1, 0)
```

## 总结一下表情键盘
- 首先一个`UIViewController`，然后子控件有两个`collectionVeiw`和`toolbar`
- `collectionVeiw` 用来显示不同的表情组，`toolbar`用来指示不同的表情组
- 然后定义模型保存表情数据 `EmoticonPackage` 根据plist文件定义字段
- 每一页显示固定数量的表情，最后一个cell添加一个删除按钮
- 最后将图片表情以富文本的附件`NSTextAttachment`形式插入到文本中
