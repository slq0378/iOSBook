# 重构_改善既有代码的设计
- 重构技术就是以微小的步伐修改程序，如果你犯下错误，很容易便可发现它。
- 作为一个傻瓜都能写出的计算机可以理解的代码，唯有写出人类容易理解的代码，才是优秀的程序员。
- 重构的节奏就是：测试、小修改、测试、小修改、测试小修改.....
- 添加新功能时，就只添加新功能，不要重构，而重构时，不要添加新功能，只改变程序结构。

## 代码的问题
- 1、`Duplicated Code`
    - 一个类中不同函数的的实现代码相同，那么要将这个函数代码抽离出一个单独的函数
    - 不同类中某两个方法代码重复，抽离到父类当中
    - 两个不相干的类中重复代码，对其中一个使用`Extract Method` 抽出一个独立类，然后在另一个类中使用这个独立类。
- 2、
- 


## 专用名词
- `Extract Method`
- `Pull Up Method`
- `Form Template Method`
- `Substitute Algorithm`
- `Replace Temp with Query` 消除临时变量
- `Introduce Parameter Object` 精简参数列表
- `Preserve Whole Object` 精简参数列表
- `Replace Method with Method Object` 继续精简参数列表和临时变量
- `Decompose Conditional` 处理条件表达式
- `Extract Subclass`
- `Extract Interface`
- `Duplicate Observed Data`
- `Replace Parameter with Method` 
- `Inlin Class` 内部类
- `Replace Data Value with Object`
- `Replace Type Code with Class`
- `Replace Type Code with Subclass`
- `Replace Type Code with State/Strategy`
- `Replace Array with Object`
- `Replace Conditional with Polymorphism`
- `Replace Parameter with Explicit Methods` 
- `Introduce Null Object`
- `Collapse Hierarchy`
- `Remove Parameter`
- `Rename Method`
- `Hide Delegate`
- `Remove Middle Man`
- `Replace Delegation with Inheritance`
- `Change Bidirectional Association to Unidirectional` 
- `Replace Inheritance with Delegation`
- `Encapsulate Collection`
- `Encapsulate Field`
- `Push Down Method`
- `Push Down Field`
- `Introduce Assertion`
- 