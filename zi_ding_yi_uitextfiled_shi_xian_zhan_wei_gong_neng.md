# 自定义UITextField（增加占位文字功能）

- 创建带有占位文字功能的textField

- 文字编辑
	- `UITextField`
		- 只能显示一行，有占位文字
	- `UITextView`
		- 可以显示多行文字，没有占位文字
- 实际需要：可以显示多行，又有占位文字
- 方法：继承UITextView，然后再增加占位文字功能
	- placeholder 实现
- drawRect实现占位文字
	- 无法随光标滚动而滚动

```objc
- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {

        self.alwaysBounceVertical = YES;
        // 监听textView的文字改变通知
        [SLQNotificationCenter addObserver:self selector:@selector(textChanged) name:UITextViewTextDidChangeNotification object:nil];
    }
    return self;
}
- (void)textChanged
{
    // 重绘
    [self setNeedsDisplay];
}

- (void)dealloc
{
    [SLQNotificationCenter removeObserver:self];
}

- (void)drawRect:(CGRect)rect
{
    // 如果输入的有文字就不显示占位文字
//    if(self.text.length || self.attributedText.length) return;
    if (self.hasText) return;
    // 设置占位文字frame
    rect.origin.x = 5;
    rect.origin.y = 7;
    rect.size.width -= 2 * rect.origin.x;
    // 绘制占位文字
    [self.placeholder drawInRect:rect withAttributes:@{NSFontAttributeName : [UIFont systemFontOfSize:15]}];
}

- (void)setPlaceholder:(NSString *)placeholder
{
    _placeholder = placeholder;
    [self setNeedsDisplay];
}
- (void)setPlaceholderColor:(UIColor *)placeholderColor
{
    _placeholderColor = placeholderColor;
    [self setNeedsDisplay];
}

```
- 添加lable实现占位文字

```	objc
@interface SLQPlaceholderText ()
/**placeholder Lable*/
@property (nonatomic, weak) UILabel *placeholderLabel;
@end

@implementation SLQPlaceholderText

- (UILabel *)placeholderLabel
{
    if (!_placeholderLabel) {
        // 添加label，实现占位文字
        UILabel *label = [[UILabel alloc] init];
        label.numberOfLines = 0;
        label.x = 5;
        label.y = 8;
        [self addSubview:label];

        _placeholderLabel = label;
    }
    return  _placeholderLabel;
}
- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {

        // 默认字体和字体颜色
        self.font = [UIFont systemFontOfSize:17];
        self.placeholderColor = [UIColor grayColor];

        self.alwaysBounceVertical = YES;
        // 监听textView的文字改变通知
        [SLQNotificationCenter addObserver:self selector:@selector(textChanged) name:UITextViewTextDidChangeNotification object:nil];
    }
    return self;
}
- (void)textChanged
{
    self.placeholderLabel.hidden = self.hasText;
}
- (void)dealloc
{
    [SLQNotificationCenter removeObserver:self];
}

/**
 *	计算占位文字高度
 */
- (void)updateTextH
{
    CGSize maxSize = CGSizeMake(SLQScreenW - 2 * self.placeholderLabel.x, MAXFLOAT);
    self.placeholderLabel.size =  [self.placeholder boundingRectWithSize:maxSize options:NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName : self.font} context:nil].size;;
}
- (void)setPlaceholder:(NSString *)placeholder
{
    _placeholder = [placeholder copy];
    self.placeholderLabel.text = placeholder;
    [self updateTextH];
}
- (void)setPlaceholderColor:(UIColor *)placeholderColor
{
    _placeholderColor = placeholderColor;
    self.placeholderLabel.textColor = placeholderColor;
}

- (void)setFont:(UIFont *)font{
    [super setFont:font];
    self.placeholderLabel.font = font;
    // 计算高度
    [self textChanged];
}
- (void)setText:(NSString *)text
{
    [super setText:text];
    [self textChanged];
}
- (void)setAttributedText:(NSAttributedString *)attributedText
{
    [super setAttributedText:attributedText];
    [self textChanged];
}
```
