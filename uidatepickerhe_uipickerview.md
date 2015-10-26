# UIDatePicker和UIPickerView

## UIDatePicker

- 这是UIDatePicker，可以显示日期和时间。



- 这个是UIPickerView，显示类似几个选择项的界面。

- 注意点:PickerView的高度不能改,默认162,PickerView里面每行的高度 可以改,不要弄混淆了。





- 做一个简单界面进行练习。单击生日输入框弹出自定义UIDatePicker，单击城市弹出自定义UIPickerView选择城市。


### UIDatePicker使用

- 在点击过文本输入框后弹出日期选中键盘。需要给UITextField控件的inutView属性指定需要显示的界面。

- 这里显示的代码如下：

```objc
- (void)setBirthdayFieldKeyboard
{
    //
    UIDatePicker *datePicker = [[UIDatePicker alloc] init];
    _datePicker = datePicker;
    // 只显示时间
    datePicker.datePickerMode = UIDatePickerModeDate;
    // 显示中文
    datePicker.locale = [NSLocalelocaleWithLocaleIdentifier:@"zh"];
    // 监听值得改变
    [datePicker addTarget:selfaction:@selector(datePickerValueChanged:) forControlEvents:UIControlEventValueChanged];
    self.birthdayField.inputView = datePicker;
}
```

- 在程序加载时进行设定

```objc
- (void)viewDidLoad {
    [superviewDidLoad];
    [selfsetBirthdayFieldKeyboard];
}
```

- 然后对UITextField的一些属性进行设置，比如说不能输入日期，只能选择显示日期。可以在以下代理方法中实现

```objc
#pragma mark - textField代理方法
// 是否允许修改填充字符串
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string
{
    returnNO;
}
```

- 当然使用之前要指定代理并遵守协议。
    - 然后再选择过时间后，同步更新到文本输入框中，这里需要监听控件的valueChanged的方法

```objc
    [datePicker addTarget:self action:@selector(datePickerValueChanged:) forControlEvents:UIControlEventValueChanged];
```

- 响应方法

```objc
- (void)datePickerValueChanged:(UIDatePicker *)datePicker
{
    NSDateFormatter *formatter = [[NSDateFormatteralloc] init];
    // 格式化日期格式
    formatter.dateFormat = @"yyyy-MM-dd";
    NSString *date = [formatter stringFromDate:datePicker.date];
    // 显示时间
    self.birthdayField.text = date;
}
```

- 最后要设置以下UITextField控件的默认值，默认显示0行0列的值

- 在改变文本输入框状态时进行设置默认值。

```objc
// 是否允许开始编辑（代理方法）
- (void)textFieldDidBeginEditing:(UITextField *)textField
{
    // 添加自定义窗口
    [selfdatePickerValueChanged:self.datePicker];
}
```
### UIPickerView使用

- 先将城市选择键盘添加到文本输入框

```objc
// 设置城市键盘
- (void)setCitiesFieldKeyboard
{
    //
    UIPickerView *picker = [[UIPickerViewalloc] init];
    _cityPicker = picker;
    picker.dataSource = self;
    picker.delegate = self;
    self.cityField.inputView = picker;
}
```

- 设置代理和数据源为控制器，实现响应的方法

```objc
#pragma mark UIPickerView 数据源
// 列数
- (NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView
{
    return 2;
}
// 某一列行数
- (NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component
{
    if (component == 0)  // 省会
    {
        return self.provinces.count;
    }
    else  // 其他城市
    {
        SLQProvince *pro = self.provinces[_proIndex];
        return pro.cities.count;
    }
}
```

- 代理方法

```objc
#pragma mark UIPickerView 代理
// 每行的标题
- (NSString *)pickerView:(UIPickerView *)pickerView titleForRow:(NSInteger)row forComponent:(NSInteger)component
{
    if (component == 0)  // 省会
    {
        // 获取省会
        SLQProvince *pro = self.provinces[row];
        return pro.name;
    }
    else  // 其他城市
    {

        SLQProvince *pro = self.provinces[_proIndex];
        return pro.cities[row];
    }
}
// 是否选中某行
- (void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component
{
    // 滚动省会刷新城市
    if(component == 0)
    {
        // 记录当前省会
        _proIndex = [pickerView selectedRowInComponent:0];
        [pickerView reloadComponent:1];
    }

    // 获取选中省会
    SLQProvince *pro = self.provinces[_proIndex];
    NSInteger cityIndex = [pickerView selectedRowInComponent:1];
    NSString *cityName = pro.cities[cityIndex];
    _cityField.text = [NSString stringWithFormat:@"%@-%@",pro.name,cityName];
}
```

- 然后在文本输入框的代理方法中添加默认显示对象

```objc
// 是否允许开始编辑
- (void)textFieldDidBeginEditing:(UITextField *)textField
{
    if (textField == self.birthdayField)
    {
        // 添加自定义窗口
        [selfdatePickerValueChanged:self.datePicker];
    }
    else
    {
        [selfpickerView:self.cityPickerdidSelectRow:0inComponent:0]; // 默认显示0行0列
    }
}
```
- 效果


