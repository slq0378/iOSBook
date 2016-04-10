# Python学习


# Python
- Python是一种解释型、面向对象、动态数据类型的高级程序设计语言。

## 简介
- Python 是一个高层次的结合了解释性、编译性、互动性和面向对象的脚本语言。
- Python 的设计具有很强的可读性，相比其他语言经常使用英文关键字，其他语言的一些标点符号，它具有比其他语言更有特色语法结构。
    - Python 是一种解释型语言： 这意味着开发过程中没有了编译这个环节。类似于PHP和Perl语言。
    - Python 是交互式语言： 这意味着，您可以在一个Python提示符，直接互动执行写你的程序。
    - Python 是面向对象语言: 这意味着Python支持面向对象的风格或代码封装在对象的编程技术。
    - Python 是初学者的语言：Python 对初级程序员而言，是一种伟大的语言，它支持广泛的应用程序开发，从简单的文字处理到 WWW 浏览器再到游戏。

## Python 特点
- 1.易于学习：Python有相对较少的关键字，结构简单，和一个明确定义的语法，学习起来更加简单。
- 2.易于阅读：Python代码定义的更清晰。
- 3.易于维护：Python的成功在于它的源代码是相当容易维护的。
- 4.一个广泛的标准库：Python的最大的优势之一是丰富的库，跨平台的，在UNIX，Windows和Macintosh兼容很好。
- 5.互动模式：互动模式的支持，您可以从终端输入执行代码并获得结果的语言，互动的测试和调试代码片断。
- 6.可移植：基于其开放源代码的特性，Python已经被移植（也就是使其工作）到许多平台。
- 7.可扩展：如果你需要一段运行很快的关键代码，或者是想要编写一些不愿开放的算法，你可以使用C或C++完成那部分程序，然后从你的Python程序中调用。
- 8.数据库：Python提供所有主要的商业数据库的接口。
- 9.GUI编程：Python支持GUI可以创建和移植到许多系统调用。
- 10.可嵌入: 你可以将Python嵌入到C/C++程序，让你的程序的用户获得"脚本化"的能力。

## 环境配置
- 我用的是mac，最新版mac自带python。如果没有，可以去这个网址安装。
- 可以在链接 <http://www.python.org/download/> 上下载最新版安装。

- 环境配置是否成功可以通过在控制台输入python验证

```python
song$ python
Python 2.7.10 (v2.7.10:15c95b7d81dc, May 23 2015, 09:33:12) 
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

- 以下为Python命令行参数：

```
选项	描述
-d	在解析时显示调试信息
-O	生成优化代码 ( .pyo 文件 )
-S	启动时不引入查找Python路径的位置
-V	输出Python版本号
-X	从 1.6版本之后基于内建的异常（仅仅用于字符串）已过时。
-c cmd	执行 Python 脚本，并将运行结果作为 cmd 字符串。
file	在给定的python文件执行python脚本。
```

- 运行脚本
    - ` $ python script.py`
- hello world
    - 使用函数print输出数据
    
```python
song$ python
Python 2.7.10 (v2.7.10:15c95b7d81dc, May 23 2015, 09:33:12) 
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> print "hello ，world"
hello ，world
>>>
```

## Python 中文编码
- Python 输出 "Hello, World!"，英文没有问题，但是如果你输出中文字符"你好，世界"就有可能会碰到中文编码问题。

```python
  File "test.py", line 2
SyntaxError: Non-ASCII character '\xe4' in file test.py on line 2, but no encoding declared; see http://www.python.org/peps/pep-0263.html for details
```
- 运行文件
    - 新建一个test.py文件，文件内容为
    
    ```pyhton
    #!usr/bin/python
    print "hello python";
    ```
    - 执行文件
    - `pyhton test.py`
    
- Python中默认的编码格式是 ASCII 格式，在没修改编码格式时无法正确打印汉字，所以在读取中文时会报错。
- 解决方法为只要在文件开头加入 # -*- coding: UTF-8 -*- 或者 #coding=utf-8 就行了。

```python
#!usr/bin/python
# -*- coding:UTF-8 -*- 
print "hello python";
```

## 基础

### Python 标识符
- 在python里，标识符有字母、数字、下划线组成。
- 在python中，所有标识符可以包括英文、数字以及下划线（_），但不能以数字开头。
- *python中的标识符是区分大小写的。*
- 以下划线开头的标识符是有特殊意义的。以单下划线开头（_foo）的代表不能直接访问的类属性，需通过类提供的接口进行访问，不能用"from xxx import *"而导入；
- 以双下划线开头的（__foo）代表类的私有成员；以双下划线开头和结尾的（__foo__）代表python里特殊方法专用的标识，如__init__（）代表类的构造函数。

### Python保留字符

- 下面的列表显示了在Python中的保留字。这些保留字不能用作常数或变数，或任何其他标识符名称。
- 所有Python的关键字只包含小写字母。

|   1    |     2    |    3
|:----------:|:----------:|:----------:|
|and      |   exec   |not       |
|assert   |finally	 |or        |
|break    |for       |pass      |
|class    |from      |print     |
|continue |global    |raise     |
|def      |if        |return    |
|del      |import    |try       |
|elif     |in        |while     |
|else     |is        |with      |
|except   |lambda    |yield     |


### 行和缩进

- 学习Python与其他语言最大的区别就是，Python的代码块不使用大括号（{}）来控制类，函数以及其他逻辑判断。python最具特色的就是用缩进来写模块。
- 缩进的空白数量是可变的，但是所有代码块语句必须包含相同的缩进空白数量，这个必须严格执行。如下所示：

```python
if True:
    print "True"
else:
    print "False"
```

```python
#!usr/bin/python
if True:
	print"true";
else :
	print "false";
 print "ahha";
```
- 这样就会报错

```python
File "test.py", line 7
    print "ahha";
                ^
IndentationError: unindent does not match any outer indentation level
```

- `IndentationError: unexpected indent` 错误是python编译器是在告诉你"Hi，老兄，你的文件里格式不对了，可能是tab和空格没对齐的问题"，所有python对格式要求非常严格。
- 如果是 IndentationError: unindent does not match any outer indentation level错误表明，你使用的缩进方式不一致，有的是 tab 键缩进，有的是空格缩进，改为一致即可。
- 因此，在Python的代码块中必须使用相同数目的行首缩进空格数。
    - 建议你在每个缩进层次使用 单个制表符 或 两个空格 或 四个空格 , 切记不能混用

### 多行语句

- Python语句中一般以新行作为为语句的结束符。
- 但是我们可以使用斜杠（ \）将一行的语句分为多行显示，如下所示：

```python
	total = 10 + \
	 10 + \
	 20;
	print total;
```

- 语句中包含[], {} 或 () 括号就不需要使用多行连接符。如下实例：

```python
	days = ['one','two','three',
	'four'];
	print days;
```
### python引号

- Python 接收单引号(' )，双引号(" )，三引号(''' """) 来表示字符串，引号的开始与结束必须的相同类型的。
- 其中三引号可以由多行组成，编写多行文本的快捷语法，常用语文档字符串，在文件的特定地点，被当做注释。

```python
	word = '哈哈哈\nss'
	sentens = "橘子"
	paragraph = """段落，尅换行
	韩哈哈哈哈哈哈哈哈哈哈哈哈 """
	print word,sentens,paragraph;
```

### Python注释

- python中单行注释采用 # 开头

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
# 文件名：test.py

# 第一个注释
print "Hello, Python!";  # 第二个注释
```

- python 中多行注释使用三个单引号(''')或三个双引号(""")。

```python

'''
这是多行注释，使用单引号。
这是多行注释，使用单引号。
'''

```

### Python空行
- 函数之间或类的方法之间用空行分隔，表示一段新的代码的开始。类和函数入口之间也用一行空行分隔，以突出函数入口的开始。
- 空行与代码缩进不同，空行并不是Python语法的一部分。书写时不插入空行，Python解释器运行也不会出错。但是空行的作用在于分隔两段不同功能或含义的代码，便于日后代码的维护或重构。
- 记住：空行也是程序代码的一部分。

### 等待用户输入
- 下面的程序在按回车键后就会等待用户输入：

```python
#!usr/bin/python
# -*- coding:UTF-8 -*-  
raw_input("\n\nPress the enter key to exit.")
```

- 以上代码中 ，"\n\n"在结果输出前会输出两个新的空行。一旦用户按下键时，程序将退出。

### 同一行显示多条语句
- Python可以在同一行中使用多条语句，语句之间使用分号(;)分割，以下是一个简单的实例：
- `print "hello"; print 'world';`

### 多个语句构成代码组
- 缩进相同的一组语句构成一个代码块，我们称之代码组。
- 像if、while、def和class这样的复合语句，首行以关键字开始，以冒号( : )结束，该行之后的一行或多行代码构成代码组。
- 我们将首行及后面的代码组称为一个子句(clause)。
- 如下实例：

```python
if expression : 
   suite 
elif expression :  
   suite  
else :  
   suite 
```

### 命令行参数

```python
$ python -h
usage: /Library/Frameworks/Python.framework/Versions/2.7/Resources/Python.app/Contents/MacOS/Python [option] ... [-c cmd | -m mod | file | -] [arg] ...
Options and arguments (and corresponding environment variables):
-B     : don't write .py[co] files on import; also PYTHONDONTWRITEBYTECODE=x
-c cmd : program passed in as string (terminates option list)
-d     : debug output from parser; also PYTHONDEBUG=x
-E     : ignore PYTHON* environment variables (such as PYTHONPATH)
-h     : print this help message and exit (also --help)
-i     : inspect interactively after running script; forces a prompt even
         if stdin does not appear to be a terminal; also PYTHONINSPECT=x
-m mod : run library module as a script (terminates option list)
-O     : optimize generated bytecode slightly; also PYTHONOPTIMIZE=x
-OO    : remove doc-strings in addition to the -O optimizations
-R     : use a pseudo-random salt to make hash() values of various types be
         unpredictable between separate invocations of the interpreter, as
         a defense against denial-of-service attacks
-Q arg : division options: -Qold (default), -Qwarn, -Qwarnall, -Qnew
-s     : don't add user site directory to sys.path; also PYTHONNOUSERSITE
-S     : don't imply 'import site' on initialization
-t     : issue warnings about inconsistent tab usage (-tt: issue errors)
-u     : unbuffered binary stdout and stderr; also PYTHONUNBUFFERED=x
         see man page for details on internal buffering relating to '-u'
-v     : verbose (trace import statements); also PYTHONVERBOSE=x
         can be supplied multiple times to increase verbosity
-V     : print the Python version number and exit (also --version)
-W arg : warning control; arg is action:message:category:module:lineno
         also PYTHONWARNINGS=arg
-x     : skip first line of source, allowing use of non-Unix forms of #!cmd
-3     : warn about Python 3.x incompatibilities that 2to3 cannot trivially fix
file   : program read from script file
-      : program read from stdin (default; interactive mode if a tty)
arg ...: arguments passed to program in sys.argv[1:]

Other environment variables:
PYTHONSTARTUP: file executed on interactive startup (no default)
PYTHONPATH   : ':'-separated list of directories prefixed to the
               default module search path.  The result is sys.path.
PYTHONHOME   : alternate <prefix> directory (or <prefix>:<exec_prefix>).
               The default module search path uses <prefix>/pythonX.X.
PYTHONCASEOK : ignore case in 'import' statements (Windows).
PYTHONIOENCODING: Encoding[:errors] used for stdin/stdout/stderr.
PYTHONHASHSEED: if this variable is set to 'random', the effect is the same
   as specifying the -R option: a random value is used to seed the hashes of
   str, bytes and datetime objects.  It can also be set to an integer
   in the range [0,4294967295] to get hash values with a predictable seed.
```

## Python 变量类型
- 变量存储在内存中的值。这就意味着在创建变量时会在内存中开辟一个空间。
- 基于变量的数据类型，解释器会分配指定内存，并决定什么数据可以被存储在内存中。
- 因此，变量可以指定不同的数据类型，这些变量可以存储整数，小数或字符。

### 变量赋值
- Python中的变量不需要声明，变量的赋值操作既是变量声明和定义的过程。
- 每个变量在内存中创建，都包括变量的标识，名称和数据这些信息。
- 每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。
- 等号（=）用来给变量赋值。
- 等号（=）运算符左边是一个变量名,等号（=）运算符右边是存储在变量中的值。例如：

```python
floatNum = 100.023 # 浮点型
intNum = 2122 # 整形
name = "jhone" # 字符串

print floatNum,intNum,name
```
### 多个变量赋值
- Python允许你同时为多个变量赋值。例如：
    -   `a = b = c = 1`
- 以上实例，创建一个整型对象，值为1，三个变量被分配到相同的内存空间上。
- 您也可以为多个对象指定多个变量。例如：
    - `a, b, c = 1, 2, "john"`
- 以上实例，两个整型对象1和2的分配给变量a和b，字符串对象"john"分配给变量c。

### 标准数据类型
- 在内存中存储的数据可以有多种类型。
- 例如，person.s年龄作为一个数值存储和他或她的地址是字母数字字符存储。
- Python有一些标准类型用于定义操作上，他们和为他们每个人的存储方法可能。
- Python有五个标准的数据类型：
    - Numbers（数字）
    - String（字符串）
    - List（列表）
    - Tuple（元组）
    - Dictionary（字典）

### Python数字
- 数字数据类型用于存储数值。
- 他们是不可改变的数据类型，这意味着改变数字数据类型会分配一个新的对象。
- 当你指定一个值时，Number对象就会被创建：

```python
var1 = 1
var2 = 10
```

- 您也可以使用del语句删除一些对象引用。
- del语句的语法是：

```python
del var1[,var2[,var3[....,varN]]]]
```

- 您可以通过使用del语句删除单个或多个对象。例如：

```python
del var
del var_a, var_b
```

- Python支持四种不同的数值类型：
    - int（有符号整型）
    - long（长整型[也可以代表八进制和十六进制]）
    - float（浮点型）
    - complex（复数）
- 长整型也可以使用小写"L"，但是还是建议您使用大写"L"，避免与数字"1"混淆。Python使用"L"来显示长整型。
- Python还支持复数，复数由实数部分和虚数部分构成，可以用a + bj,或者complex(a,b)表示， 复数的实部a和虚部b都是浮点型

### Python字符串
- 字符串或串(String)是由数字、字母、下划线组成的一串字符。
- 一般记为 :
    - `s="a1a2···an"(n>=0)`
- 它是编程语言中表示文本的数据类型。
- python的字串列表有2种取值顺序:
    - 从左到右索引默认0开始的，最大范围是字符串长度少1
    - 从右到左索引默认-1开始的，最大范围是字符串开头
- 如果你的实要取得一段子串的话，可以用到变量[头下标:尾下标]，就可以截取相应的字符串，其中下标是从0开始算起，可以是正数或负数，下标可以为空表示取到头或尾。
- 比如:
    - `s = 'ilovepython'`
    - `s[1:5]的结果是love。`
- 当使用以冒号分隔的字符串，python返回一个新的对象，结果包含了以这对偏移标识的连续的内容，左边的开始是包含了下边界。
- 上面的结果包含了s[1]的值l，而取到的最大范围不包括上边界，就是s[5]的值p。
加号（+）是字符串连接运算符，星号（*）是重复操作。如下实例：

```python
name = "hello world!" # 字符串

print name # 完整字符串
print name[0] # 第一个字符
print name[2:5] # 从2-5的字符串
print name[2:] # 从2开会的字符串
print name * 2 # 重复输出
print name + name # 拼接字符串
```

### Python列表
- List（列表） 是 Python 中使用最频繁的数据类型。
- 列表可以完成大多数集合类的数据结构实现。它支持字符，数字，字符串甚至可以包含列表（所谓嵌套）。
- 列表用[ ]标识。是python最通用的复合数据类型。看这段代码就明白。
- 列表中的值得分割也可以用到变量[头下标:尾下标]，就可以截取相应的列表，从左到右索引默认0开始的，从右到左索引默认-1开始，下标可以为空表示取到头或尾。
- 加号（+）是列表连接运算符，星号（*）是重复操作。如下实例：

```python
list = ["1","2","3","4","5"]
name = ["zhang","wang","li"]

print list
print list[1]
print list[1:]
print list[2:4]
print list * 2
print list + name
```

### Python元组
- 元组是另一个数据类型，类似于List（列表）。
- 元组用"()"标识。内部元素用逗号隔开。但是元素不能二次赋值，相当于*只读列表*。

```python
tuple = ("1","2","3","4","5")
name = ("zhang","wang","li")

print tuple
print tuple[0]
print tuple[1:]
print tuple[2:4]
print tuple * 2
print tuple + name
print name[1]
```

### Python元字典
- 字典(dictionary)是除列表以外python之中最灵活的内置数据结构类型。列表是有序的对象结合，字典是无序的对象集合。
- 两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。
字典用"{ }"标识。字典由索引(key)和它对应的值value组成。

```python
dict = {"name":"ZhangSan","age":"23","dept":"computer"}

print dict #输出整个字典
print dict["name"] # 输出指定key的value
print dict.keys() # 输出所有的key
print dict.values() # 输出所有的value
```

### Python数据类型转换
- 有时候，我们需要对数据内置的类型进行转换，数据类型的转换，你只需要将数据类型作为函数名即可。
- 以下几个内置的函数可以执行数据类型之间的转换。这些函数返回一个新的对象，表示转换的值。

        函数      |        描述
|:--------------------- |:--------------------|
int(x[,base])           |   将x转换为一个整数
long(x[,base])          |   将x转换为一个长整数
float(x)                |   将x转换到一个浮点数
complex(real[,imag])    |   创建一个复数
str(x)                  |   将对象 x 转换为字符串
repr(x)                 |   将对象 x 转换为表达式字符串
eval(str)               |   用来计算在字符串中的有效Python表达式,并返回一个对象
tuple(s)                |   将序列 s 转换为一个元组
list(s)                 |   将序列 s 转换为一个列表
set(s)                  |   转换为可变集合
dict(d)                 |   创建一个字典。d 必须是一个序列 (key,value)元组。
frozenset(s)            |   转换为不可变集合
chr(x)                  |   将一个整数转换为一个字符
unichr(x)               |   将一个整数转换为Unicode字符
ord(x)                  |   将一个字符转换为它的整数值
hex(x)                  |   将一个整数转换为一个十六进制字符串
oct(x)                  |   将一个整数转换为一个八进制字符串


```python
intNum = 100
floatNum = 100.1
print "int" ,int(floatNum)
print "float" ,float(intNum)
print "long" , long(intNum)
print "string", str(intNum)
print "complex", complex(intNum)
print "repr",repr(intNum)
print "eval",eval(str(floatNum))
print "tuple", tuple(["1","2"])
print "list", list(["1","2"])
print "set",set(["1","2","name"])
print "frozenset",frozenset(["123","hahah"])
print "chr",chr(100)
print "unichr",unichr(100)
print "ord", ord('1') 
print "hex", hex(100)
print "oct", oct(100)
```

## Python 运算符

- Python语言支持以下类型的运算符:
    - 算术运算符
    - 比较（关系）运算符
    - 赋值运算符 
    - 逻辑运算符 (and, or, not)
    - 位运算符 (&, |, ^, ~, <<, >>)
    - 成员运算符 (in , not in)
    - 身份运算符
    - 运算符优先级

- 这里记录一下和C不一样的地方，这里有个整除符号，是双斜杠 "//" ,结果取商的整数部分

```python
print int(11.5/2) # 5,没有进行四舍五入，直接去除小数点后数字
print int(11.5//2) # 5，直接取整数部分，不考虑小树部分
```

- `**=	幂赋值运算符	c **= a 等效于 c = c ** a`

```python
num = 2
print  # 2
num **= 3 # num 的3次幂
print num # 8
```
### Python成员运算符
- 除了以上的一些运算符之外，Python还支持成员运算符，测试实例中包含了一系列的成员，包括字符串，列表或元组。
- in	
    - 如果在指定的序列中找到值返回True，否则返回False。 
    - x 在 y序列中 , 如果x在y序列中返回True。
- not in	
    - 如果在指定的序列中没有找到值返回True，否则返回False。
    - x 不在 y序列中 , 如果x不在y序列中返回True。

```python
num1 = 2
num2 = 2
num3 = 3
print num1 is num2 # true
print num1 is num3 # false
```

## Python 条件语句
- Python程序语言指定任何非0和非空（null）值为true，0 或者 null为false。
- 由于 python 并不支持 switch 语句，所以多个条件判断，只能用 elif 来实现，如果判断需要多个条件需同时判断时，可以使用 or （或），表示两个条件有一个成立时判断条件成功；使用 and （与）时，表示只有两个条件同时成立的情况下，判断条件才成功。
- 当if有多个条件时可使用括号来区分判断的先后顺序，括号中的判断优先执行，此外 and 和 or 的优先级低于>（大于）、<（小于）等判断符号，即大于和小于在没有括号的情况下会比与或要优先判断。
- 简单的语句组
    - 你也可以在同一行的位置上使用if条件判断语句，如下实例：

```python
#!/usr/bin/python 
# -*- coding: UTF-8 -*-
 
var = 100 
 
if ( var  == 100 ) : print "变量 var 的值为100" 
 
print "Good bye!" 
```

## Python 循环语句
- Python提供了for循环和while循环（在Python中没有do..while循环）
- 循环控制语句可以更改语句执行的顺序。Python支持以下循环控制语句：
    - break 语句	在语句块执行过程中终止循环，并且跳出整个循环
    - continue 语句	在语句块执行过程中终止当前循环，跳出该次循环，执行下一次循环。
    - pass 语句	pass是空语句，是为了保持程序结构的完整性。

- 循环使用 else 语句
- 在 python 中，for … else 表示这样的意思，for 中的语句和普通的没有区别，else 中的语句会在循环正常执行完（即 for 不是通过 break 跳出而中断的）的情况下执行，while … else 也是一样。

```python
#!/usr/bin/python

count = 0
while count < 5:
   print count, " is  less than 5"
   count = count + 1
else:
   print count, " is not less than 5"
```

- 简单语句组
类似if语句的语法，如果你的while循环体中只有一条语句，你可以将该语句与while写在同一行中， 如下所示：

```python
#!/usr/bin/python

flag = 1

while (flag): print 'Given flag is really true!'

print "Good bye!"
```

- for循环

```python
for iterating_var in sequence:
   statements(s)
```

- 通过序列索引迭代
    - 另外一种执行循环的遍历方式是通过索引，如下实例：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

fruits = ['banana', 'apple',  'mango']
for index in range(len(fruits)):
   print '当前水果 :', fruits[index]

print "Good bye!"
```

- 内置函数 len() 和 range(),函数 len() 返回列表的长度，即元素的个数。 range返回一个序列的数。

- 循环使用 else 语句
    - 在 python 中，for … else 表示这样的意思，for 中的语句和普通的没有区别，else 中的语句会在循环正常执行完（即 for 不是通过 break 跳出而中断的）的情况下执行，while … else 也是一样。

- Python pass 语句
    - Python pass是空语句，是为了保持程序结构的完整性。
    - passass 不做任何事情，一般用做占位语句。

## Python数学函数

    函数      |   	返回值 ( 描述 )
|:------------------------------|:------------------------------|
abs(x)	|返回数字的绝对值，如abs(-10) 返回 10
ceil(x)	|返回数字的上入整数，如math.ceil(4.1) 返回 5
cmp(x,y)	|如果 x < y 返回 -1, 如果 x == y 返回 0, 如果 x > y 返回 1
exp(x)	|返回e的x次幂(ex),如math.exp(1) 返回2.718281828459045
fabs(x)	|返回数字的绝对值，如math.fabs(-10) 返回10.0
floor(x)	|返回数字的下舍整数，如math.floor(4.9)返回 4
log(x)	|如math.log(math.e)返回1.0,math.log(100,10)返回2.0
log10(x)	|返回以10为基数的x的对数，如math.log10(100)返回 2.0
max(x1,x2,...)	|返回给定参数的最大值，参数可以为序列。
min(x1,x2,...)	|返回给定参数的最小值，参数可以为序列。
modf(x)	|返回x的整数部分与小数部分，两部分的数值符号与x相同，整数部分以浮点型表示。
pow(x,y)	|x**y 运算后的值。
round(x[,n])	|返回浮点数x的四舍五入值，如给出n值，则代表舍入到小数点后的位数。
sqrt(x)	|返回数字x的平方根，数字可以为负数，返回类型为实数，如math.sqrt(4)返回 2+0j

## Python随机数函数
- 随机数可以用于数学，游戏，安全等领域中，还经常被嵌入到算法中，用以提高算法效率，并提高程序的安全性。
- Python包含以下常用随机数函数：

    函数	  |   描述
|:-------------------|:-------------------|
choice(seq)	    |从序列的元素中随机挑选一个元素，比如random.choice(range(10))，从0到9中随机挑选一个整数。
randrange ([start,]stop[,step])	  |从指定范围内，按指定基数递增的集合中获取一个随机数，基数缺省值为1
random()    	|随机生成下一个实数，它在[0,1)范围内。
seed([x])	  |改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。
shuffle(lst)	   |将序列的所有元素随机排序
uniform(x, y)   |随机生成下一个实数，它在[x,y]范围内。


## Python三角函数
- Python包括以下三角函数：

    函数	|    描述
|:----------:|:-----------|
acos(x)	|返回x的反余弦弧度值。
asin(x)	|返回x的反正弦弧度值。	
atan(x)	|返回x的反正切弧度值。
atan2(y, x)	|返回给定的 X 及 Y 坐标值的反正切值。
cos(x)	|返回x的弧度的余弦值。
hypot(x, y)	|返回欧几里德范数 sqrt(x*x + y*y)。
sin(x)	|返回的x弧度的正弦值。
tan(x)	|返回x弧度的正切值。
degrees(x)	|将弧度转换为角度,如degrees(math.pi/2) ， 返回90.0
radians(x)	|将角度转换为弧度

## Python数学常量

    常量	|    描述
|:-----------|:-----------|
pi	数学常量 |   pi（圆周率，一般以π来表示）
e	数学常量 |    e，e即自然常数（自然常数）。

## Python字符串 
- Python转义字符
- 在需要在字符中使用特殊字符时，python用反斜杠(\)转义字符。如下表：

    转义字符	|  描述
|:------------:|:-----------|
\ (在行尾时)|	续行符
\\  |反斜杠符号
\'	|单引号
\"	|双引号
\a	|响铃
\b	|退格(Backspace)
\e	|转义
\000	|空
\n	|换行
\v	|纵向制表符
\t	|横向制表符
\r	|回车
\f	|换页
\oyy	|八进制数，yy代表的字符，例如：\o12代表换行
\xyy	|十六进制数，yy代表的字符，例如：\x0a代表换行
\other	|其它的字符以普通格式输出

## Python字符串运算符
- 下表实例变量a值为字符串"Hello"，b变量值为"Python"：


|   操作符	         |       描述	|    实例  |
|:--------------:|:------------|:----------|
+	|字符串连接	|a + b 输出结果： HelloPython
*	|重复输出字符串	|a*2 输出结果：HelloHello
[1000000]	|通过索引获取字符串中字符	|a[1] 输出结果 e
[:]	|截取字符串中的一部分	|a[1:4] 输出结果 ell
in	|成员运算符 - 如果字符串中包含给定的字符返回 True	|H in a 输出结果 1
not in	|成员运算符 - 如果字符串中不包含给定的字符返回 True	|M not in a 输出结果 1

-  r/R运算符	
    - 原始字符串 - 原始字符串：所有的字符串都是直接按照字面的意思来使用，没有转义特殊或不能打印的字符。 原始字符串除在字符串的第一个引号前加上字母"r"（可以大小写）以外，与普通字符串有着几乎完全相同的语法。
    - 	print r'\n' prints \n 和 print R'\n' prints \n

## Python字符串格式化
- Python 支持格式化字符串的输出 。尽管这样可能会用到非常复杂的表达式，但最基本的用法是将一个值插入到一个有字符串格式符 %s 的字符串中。
- 在 Python 中，字符串格式化使用与 C 中 sprintf 函数一样的语法。
如下实例：

```python
#!/usr/bin/python
print "My name is %s and weight is %d kg!" % ('Zara', 21) 
```

### python字符串格式化符号:
  
|    符号	    |         描述     |
|:------------:|:---------------|
      %c	 |格式化字符及其ASCII码
      %s	 |格式化字符串
      %d	 |格式化整数
      %u	 |格式化无符号整型
      %o	 |格式化无符号八进制数
      %x	 |格式化无符号十六进制数
      %X	 |格式化无符号十六进制数（大写）
      %f	 |格式化浮点数字，可指定小数点后的精度
      %e	 |用科学计数法格式化浮点数
      %E	 |作用同%e，用科学计数法格式化浮点数
      %g	 |%f和%e的简写
      %G	 |%f 和 %E 的简写
      %p	 |用十六进制数格式化变量的地址

### 格式化操作符辅助指令:

|      符号	|    功能  |
|:--------:|:----------|
*	|定义宽度或者小数点精度
-	|用做左对齐
+	|在正数前面显示加号( + )
<sp>	|在正数前面显示空格
\#	|在八进制数前面显示零('0')，在十六进制前面显示'0x'或者'0X'(取决于用的是'x'还是'X')
0	|显示的数字前面填充'0'而不是默认的空格
%	|'%%'输出一个单一的'%'
(var)	|映射变量(字典参数)
m.n.	|m 是显示的最小总宽度,n 是小数点后的位数(如果可用的话)

## Unicode 字符串
- Python 中定义一个 Unicode 字符串和定义一个普通字符串一样简单：

```python
>>> u'Hello World !'
u'Hello World !'
```
- 引号前小写的"u"表示这里创建的是一个 Unicode 字符串。如果你想加入一个特殊字符，可以使用 Python 的 Unicode-Escape 编码。如下例所示：

```python
>>> u'Hello\u0020World !'
u'Hello World !'
```

- 被替换的 \u0020 标识表示在给定位置插入编码值为 0x0020 的 Unicode 字符（空格符）。

## Python 列表(Lists)

### Python包含以下函数:

|   序号	|    函数      |
|:--------|:---------|
    1	|cmp(list1, list2) 比较两个列表的元素
    2	|len(list)列表元素个数
    3	|max(list)返回列表元素最大值
    4	|min(list)返回列表元素最小值
    5	|list(seq)将元组转换为列表

### Python包含以下方法:

|    序号	        |       方法         |
|:--------------: |:------------------|
    0001	|list.append(obj)在列表末尾添加新的对象
    02	|list.count(obj)统计某个元素在列表中出现的次数
    03	|list.extend(seq)在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表）
    04	|list.index(obj)从列表中找出某个值第一个匹配项的索引位置
    05	|list.insert(index, obj)将对象插入列表
    06	|list.pop(obj=list[-1])移除列表中的一个元素（默认最后一个元素），并且返回该元素的值
    07	|list.remove(obj)移除列表中某个值的第一个匹配项
    08	|list.reverse()反向列表中元素
    09	|list.sort([func])对原列表进行排序

## Python 元组
- 删除元组
    - 元组中的元素值是不允许删除的，但我们可以使用del语句来删除整个元组

```python
#!/usr/bin/python

tup = ('physics', 'chemistry', 1997, 2000);

print tup;
del tup;
print "After deleting tup : "
print tup;
```

### Python元组包含了以下内置函数
|   序号	|方法及描述|
|:--------:|:----------|
    1	|cmp(tuple1, tuple2)比较两个元组元素。
    2	|len(tuple)计算元组元素个数。
    3	|max(tuple)返回元组中元素最大值。
    4	|min(tuple)返回元组中元素最小值。
    5	|tuple(seq)将列表转换为元组。

## Python 字典(Dictionary)

### 删除字典元素

- 能删单一的元素也能清空字典，清空只需一项操作。
- 显示删除一个字典用del命令，如下实例：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

dict = {'Name': 'Zara', 'Age': 7, 'Class': 'First'};
 
del dict['Name']; # 删除键是'Name'的条目
dict.clear();     # 清空词典所有条目
del dict ;        # 删除词典
 
print "dict['Age']: ", dict['Age'];
print "dict['School']: ", dict['School'];
```
### 字典键的特性
- 字典值可以没有限制地取任何python对象，既可以是标准的对象，也可以是用户定义的，但键不行。
- 两个重要的点需要记住：
    - 1）不允许同一个键出现两次。创建时如果同一个键被赋值两次，后一个值会被记住，如下实例：
    - 2）键必须不可变，所以可以用数字，字符串或元组充当，所以用列表就不行，如下实例：

### Python字典包含了以下内置函数：

|   序号	|    函数及描述   |
|--------------|----------|
    0001	|cmp(dict1, dict2)比较两个字典元素。
    2	|len(dict)计算字典元素个数，即键的总数。
    3	|str(dict)输出字典可打印的字符串表示。
    4	|type(variable)返回输入的变量类型，如果变量是字典就返回字典类型。

### Python字典包含了以下内置方法：

|   序号	|    函数及描述   |
|------------|------------|
0001	|radiansdict.clear()删除字典内所有元素
2	|radiansdict.copy()返回一个字典的浅复制
3	|radiansdict.fromkeys()创建一个新字典，以序列seq中元素做字典的键，val为字典所有键对应的初始值
4	|radiansdict.get(key, default=None)返回指定键的值，如果值不在字典中返回default值
5	|radiansdict.has_key(key)如果键在字典dict里返回true，否则返回false
6	|radiansdict.items()以列表返回可遍历的(键, 值) 元组数组
7	|radiansdict.keys()以列表返回一个字典所有的键
8	|radiansdict.setdefault(key, default=None)和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default
9	|radiansdict.update(dict2)把字典dict2的键/值对更新到dict里
10	|radiansdict.values()以列表返回字典中的所有值

### Python 日期和时间
- Python有一个 time 和 calendar 模组可以帮忙。
- 什么是Tick？
    - 时间间隔是以秒为单位的浮点小数。
    - 每个时间戳都以自从1970年1月1日午夜（历元）经过了多长时间来表示。
    - Python附带的受欢迎的time模块下有很多函数可以转换常见日期格式。如函数time.time()用ticks计时单位返回从12:00am, January 1, 1970(epoch) 开始的记录的当前操作系统时间, 如下实例:

```python
import time;#this is required to include time module
ticks = time.time()
print "number of ticks since 12:00 am,January 1, 1970:",ticks
```

#### 什么是时间元组？
- 很多Python函数用一个元组装起来的9组数字处理时间:

|   序号	 |       字段	 |       值        |
|:---------:|:---------|:---------|
    0000   |   4位数年 |  2008
1	 |月	 |    1 到 12
2	 |日	 |1到31
3	 |小时	 |0到23
4	 |分钟  |0到60
5	 |秒	 |0到61 (60或61 是闰秒)
6	 |一周的第几日	 |0到6 (0是周一)
7	 |一年的第几日	 |1到366 (儒略历)
8	 |夏令时	 |-1, 0, 1, -1是决定是否为夏令时的旗帜


#### 上述也就是struct_time元组。
- 这种结构具有如下属性：

|   序号	|    属性	|    值   |
|:---------:|:---------|:--------|
    00000	| tm_year	| 	2008
1		| tm_mon		| 1 到 12
2		| tm_mday		| 1 到 31
3		| tm_hour		| 0 到 23
4		| tm_min		| 0 到 59
5		| tm_sec		| 0 到 61 (60或61 是闰秒)
6		| tm_wday		| 0到6 (0是周一)
7		| tm_yday		| 1 到 366(儒略历)
8		| tm_isdst		| -1, 0, 1, -1是决定是否为夏令时的旗帜

- 获取当前时间
    - 获取可读的时间模式的函数是asctime();
    
```python
import time;
localtime = time.asctime(time.localtime(time.time()))
print "localtime:",localtime
```

- 获取某月日历
    - Calendar模块有很广泛的方法用来处理年历和月历，例如打印某月的月历：

```python
import calendar;
cal = calendar.month(2008,1)
print "日历："
print cal

# 输出
song$ python test.py 
日历：
    January 2008
Mo Tu We Th Fr Sa Su
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31
```


## 函数

### 函数定义
- 你可以定义一个由自己想要功能的函数，以下是简单的规则：
- 函数代码块以def关键词开头，后接函数标识符名称和圆括号()。
- 任何传入参数和自变量必须放在圆括号中间。圆括号之间可以用于定义参数。
- 函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明。
- 函数内容以冒号起始，并且缩进。
- Return[expression]结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回 None。

### 语法

```python
def functionname( parameters ):
   "函数_文档字符串"
   function_suite
   return [expression]
```

- 默认情况下，参数值和参数名称是按函数声明中定义的的顺序匹配起来的。

```python
def printName(str):
	print "Name"
	print str
	return str

def pirntAge(age):
	print age
	return age
def printInfo(name,age):	
	printName(name)
	pirntAge(age)
	print "end"
printInfo("song",45)
```

- 按值传递参数和按引用传递参数
    - 所有参数（自变量）在Python里都是按引用传递。如果你在函数里修改了参数，那么在调用这个函数的函数里，原始的参数也被改变了。

```python
def changeName(mylist):
	mylist.append([1,2,3,4])
	print mylist
	return

mylist = [10,20,30]
changeName(mylist)
print mylist
```
### 参数
- 以下是调用函数时可使用的正式参数类型：
    - 必备参数
    - 命名参数
    - 缺省参数
    - 不定长参数

- 必备参数
    - 必备参数须以正确的顺序传入函数。调用时的数量必须和声明时的一样。
- 命名参数
    - 命名参数和函数调用关系紧密，调用方用参数的命名确定传入的参数值。你可以跳过不传的参数或者乱序传参，因为Python解释器能够用参数名匹配参数值。用命名参数调用printme()函数：
- 缺省参数
    - 调用函数时，缺省参数的值如果没有传入，则被认为是默认值。下例会打印默认的age，如果age没有被传入：
    
```python
# 必备参数
def changeName(name):
	name = "zhang"
	return name
name = "song"
result = changeName(name)
print name
print result
# 命名参数
def changeName(name,age):
	name = "zhang"
	print name,age
	return
print "命名参数:" 
changeName(age = 56,name = "song")
# 缺省参数
def changeName(name,age = 90):
	name = "zhang"
	print name,age
	return
print "缺省参数:" 
changeName(name = "song")
```    

- 不定长参数
    - 你可能需要一个函数能处理比当初声明时更多的参数。这些参数叫做不定长参数，和上述2种参数不同，声明时不会命名。基本语法如下：

```python
def functionname([formal_args,] *var_args_tuple ):
   "函数_文档字符串"
   function_suite
   return [expression]
```
- 加了星号（*）的变量名会存放所有未命名的变量参数。选择不多传参数也可。

```python
# 不定长参数
def printMessage(arg1,*vartuple):
	 "打印任何传入的参数"
	 print "输出："
	 print arg1
	 for var in vartuple:	
	 	print var	
	 return;
printMessage(10)
printMessage(450,0304,3220,34034)
```

### 匿名函数
- python 使用 lambda 来创建匿名函数。
    - lambda只是一个表达式，函数体比def简单很多。
    - lambda的主体是一个表达式，而不是一个代码块。仅仅能在lambda表达式中封装有限的逻辑进去。
    - lambda函数拥有自己的名字空间，且不能访问自有参数列表之外或全局名字空间里的参数。
    - 虽然lambda函数看起来只能写一行，却不等同于C或C++的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率。
    - 语法
    - lambda函数的语法只包含一个语句，如下：
    - `lambda [arg1 [,arg2,.....argn]]:expression`

```python
# labda 表达式
sum = lambda arg1,arg2:arg1 + arg2; # 分号不要忘记
print sum( 10, 20)
```

### 变量作用域
- 一个程序的所有的变量并不是在哪个位置都可以访问的。访问权限决定于这个变量是在哪里赋值的。
- 变量的作用域决定了在哪一部分程序你可以访问哪个特定的变量名称。两种最基本的变量作用域如下：
    - 全局变量
    - 局部变量
- 调用函数时，所有在函数内声明的变量名称都将被加入到作用域中。

## Python 模块
- 模块让你能够有逻辑地组织你的Python代码段。
- 把相关的代码分配到一个 模块里能让你的代码更好用，更易懂。
- 模块也是Python对象，具有随机的名字属性用来绑定或引用。
- 简单地说，模块就是一个保存了Python代码的文件。模块能定义函数，类和变量。模块里也能包含可执行的代码。

### import 语句
- 想使用Python源文件，只需在另一个源文件里执行import语句，语法如下：
    - `import module1[, module2[,... moduleN]`
    - 当解释器遇到import语句，如果模块在当前的搜索路径就会被导入。
    - 搜索路径是一个解释器会先进行搜索的所有目录的列表。如想要导入模块hello.py，需要把命令放在脚本的顶端：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
# 导入模块
import support
 
# 现在可以调用模块里包含的函数了
support.print_func("Zara")
```

- 以上实例输出结果：
    - Hello : Zara
- 一个模块只会被导入一次，不管你执行了多少次import。这样可以防止导入模块被一遍又一遍地执行。

### From…import 语句
- Python的from语句让你从模块中导入一个指定的部分到当前命名空间中。语法如下：
    - `from modname import name1[, name2[, ... nameN]]`
- 例如，要导入模块fib的fibonacci函数，使用如下语句：
    - `from fib import fibonacci`
- 这个声明不会把整个fib模块导入到当前的命名空间中，它只会将fib里的fibonacci单个引入到执行这个声明的模块的全局符号表。

### From…import* 语句
- 把一个模块的所有内容全都导入到当前的命名空间也是可行的，只需使用如下声明：
    - `from modname import *`
- 这提供了一个简单的方法来导入一个模块中的所有项目。然而这种声明不该被过多地使用。

### 定位模块
- 当你导入一个模块，Python解析器对模块位置的搜索顺序是：
    - 当前目录
    - 如果不在当前目录，Python 则搜索在 shell 变量 PYTHONPATH 下的每个目录。
    - 如果都找不到，Python会察看默认路径。UNIX下，默认路径一般为/usr/local/lib/python/。
- 模块搜索路径存储在system模块的sys.path变量中。变量里包含当前目录，PYTHONPATH和由安装过程决定的默认目录。

### PYTHONPATH变量
- 作为环境变量，PYTHONPATH由装在一个列表里的许多目录组成。PYTHONPATH的语法和shell变量PATH的一样。
- 在Windows系统，典型的PYTHONPATH如下：
    - `set PYTHONPATH=c:\python20\lib;`
- 在UNIX系统，典型的PYTHONPATH如下：
    - `set PYTHONPATH=/usr/local/lib/python`

### 命名空间和作用域
- 变量是拥有匹配对象的名字（标识符）。命名空间是一个包含了变量名称们（键）和它们各自相应的对象们（值）的字典。
- 一个Python表达式可以访问局部命名空间和全局命名空间里的变量。如果一个局部变量和一个全局变量重名，则局部变量会覆盖全局变量。
- 每个函数都有自己的命名空间。类的方法的作用域规则和通常函数的一样。
- Python会智能地猜测一个变量是局部的还是全局的，它假设任何在函数内赋值的变量都是局部的。
因此，如果要给全局变量在一个函数里赋值，必须使用global语句。
- `global VarName`的表达式会告诉Python， VarName是一个全局变量，这样Python就不会在局部命名空间里寻找这个变量了。
- 例如，我们在全局命名空间里定义一个变量money。我们再在函数内给变量money赋值，然后Python会假定money是一个局部变量。然而，我们并没有在访问前声明一个局部变量money，结果就是会出现一个UnboundLocalError的错误。取消global语句的注释就能解决这个问题。

```python
# 全部变量
Money = 1000
def addMoney():
	global Money;
	Money += 1;	
	print Money

print Money;
addMoney();
print Money;
```

### dir()函数
- dir()函数一个排好序的字符串列表，内容是一个模块里定义过的名字。
- 返回的列表容纳了在一个模块里定义的所有模块，变量和函数。如下一个简单的实例：

```python
import math

print dir(math)

song$ python test.py 
['__doc__', '__file__', '__name__', '__package__', 'acos', 'acosh', 'asin', 'asinh', 'atan', 'atan2', 'atanh', 'ceil', 'copysign', 'cos', 'cosh', 'degrees', 'e', 'erf', 'erfc', 'exp', 'expm1', 'fabs', 'factorial', 'floor', 'fmod', 'frexp', 'fsum', 'gamma', 'hypot', 'isinf', 'isnan', 'ldexp', 'lgamma', 'log', 'log10', 'log1p', 'modf', 'pi', 'pow', 'radians', 'sin', 'sinh', 'sqrt', 'tan', 'tanh', 'trunc']
```
- 在这里，特殊字符串变量__name__指向模块的名字，__file__指向该模块的导入文件名。

### globals()和locals()函数
- 根据调用地方的不同，globals()和locals()函数可被用来返回全局和局部命名空间里的名字。
- 如果在函数内部调用locals()，返回的是所有能在该函数里访问的命名。
- 如果在函数内部调用globals()，返回的是所有在该函数里能访问的全局名字。
- 两个函数的返回类型都是字典。所以名字们能用keys()函数摘取。

### reload()函数
- 当一个模块被导入到一个脚本，模块顶层部分的代码只会被执行一次。
- 因此，如果你想重新执行模块里顶层部分的代码，可以用reload()函数。该函数会重新导入之前导入过的模块。语法如下：
    - `reload(module_name)`
- 在这里，module_name要直接放模块的名字，而不是一个字符串形式。比如想重载hello模块，如下：
    - `reload(hello)`

### Python中的包
- 包是一个分层次的文件目录结构，它定义了一个由模块及子包，和子包下的子包等组成的Python的应用环境。

## Python 文件I/O

### 打印到屏幕
- 最简单的输出方法是用print语句，你可以给它传递零个或多个用逗号隔开的表达式。此函数把你传递的表达式转换成一个字符串表达式，并将结果写到标准输出如下：`print()`

### 读取键盘输入
- Python提供了两个内置函数从标准输入读入一行文本，默认的标准输入是键盘。如下：
    - raw_input
    - input
- raw_input函数
- raw_input([prompt]) 函数从标准输入读取一个行，并返回一个字符串（去掉结尾的换行符）：

```python
str = raw_input("请输入：")
print str
```

- input函数
    - `input([prompt])` 函数和 `raw_input([prompt])` 函数基本类似，但是 input 可以接收一个Python表达式作为输入，并将运算结果返回。

### 打开和关闭文件
- 读写实际的数据文件。
- Python 提供了必要的函数和方法进行默认情况下的文件基本操作。你可以用 file 对象做大部分的文件操作。
    - `open 函数`
- 你必须先用Python内置的open()函数打开一个文件，创建一个file对象，相关的方法才可以调用它进行读写。
语法：`file object = open(file_name [, access_mode][, buffering])`

- 各个参数的细节如下：
    - file_name：file_name变量是一个包含了你要访问的文件名称的字符串值。
    - access_mode：access_mode决定了打开文件的模式：只读，写入，追加等。所有可取值见如下的完全列表。这个参数是非强制的，默认文件访问模式为只读(r)。
    - buffering:如果buffering的值被设为0，就不会有寄存。如果buffering的值取1，访问文件时会寄存行。如果将buffering的值设为大于1的整数，表明了这就是的寄存区的缓冲大小。如果取负值，寄存区的缓冲大小则为系统默认。

### File对象的属性
- 一个文件被打开后，你有一个file对象，你可以得到有关该文件的各种信息。
- 以下是和file对象相关的所有属性的列表：

|   属性	|    描述  |
|:---------|:------------|
file.closed	|   返回true如果文件已被关闭，否则返回false。
file.mode	|     返回被打开文件的访问模式。
file.name	|     返回文件的名称。
file.softspace	|    如果用print输出后，必须跟一个空格符，则返回false。否则返回true。


```python
# 读取文件
fo = open("foo.text","wb")
print "name:",fo.name
print "closed ornot：",fo.closed
print "mode:",fo.mode
print "softspace:",fo.softspace
```

### close()方法
- File 对象的 close（）方法刷新缓冲区里任何还没写入的信息，并关闭该文件，这之后便不能再进行写入。
- 当一个文件对象的引用被重新指定给另一个文件时，Python 会关闭之前的文件。用 close（）方法关闭文件是一个很好的习惯。
- 语法：`fileObject.close();`

### write()方法
- write()方法可将任何字符串写入一个打开的文件。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。
- Write()方法不在字符串的结尾不添加换行符('\n')：
语法：`fileObject.write(string);`
- 在这里，被传递的参数是要写入到已打开文件的内容。

```python
# 打开一个文件
fo = open("foo.txt", "wb")
fo.write( "www.runoob.com!\nVery good site!\n");
# 关闭打开的文件
fo.close()
```

### read()方法
- read（）方法从一个打开的文件中读取一个字符串。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。
- 语法：`fileObject.read([count]);`
- 在这里，被传递的参数是要从已打开文件中读取的字节计数。该方法从文件的开头开始读入，如果没有传入count，它会尝试尽可能多地读取更多的内容，很可能是直到文件的末尾。

```python
# 打开一个文件
fo = open("foo.txt", "r+")
str = fo.read(10);
print "read:",str
# 关闭打开的文件
fo.close()
```

### 文件定位
- tell()方法告诉你文件内的当前位置；换句话说，下一次的读写会发生在文件开头这么多字节之后。
- seek（offset [,from]）方法改变当前文件的位置。Offset变量表示要移动的字节数。From变量指定开始移动字节的参考位置。
- 如果from被设为0，这意味着将文件的开头作为移动字节的参考位置。如果设为1，则使用当前的位置作为参考位置。如果它被设为2，那么该文件的末尾将作为参考位置。
- 例子：
    - 就用我们上面创建的文件foo.txt。

```python
# 打开一个文件
fo = open("foo.txt", "r+")
str = fo.read(10);
print "read:",str
position = fo.tell(); # 查找当前位置
print "current:",position
position = fo.seek(0,0);
str = fo.read(10)
print "read:",str
# 关闭打开的文件
fo.close()
```

### 重命名和删除文件
- Python的os模块提供了帮你执行文件处理操作的方法，比如重命名和删除文件。
- 要使用这个模块，你必须先导入它，然后才可以调用相关的各种功能。
- rename()方法：
    - rename()方法需要两个参数，当前的文件名和新文件名。
- 语法：
    - `os.rename(current_file_name, new_file_name)`
- remove()方法
    - 你可以用remove()方法删除文件，需要提供要删除的文件名作为参数。

```python
import os # 重命名文件和删除文件
# os.rename("foo.txt","fool.txt")
os.remove("fool.txt")
```

### Python里的目录：
- 所有文件都包含在各个不同的目录下，不过Python也能轻松处理。os模块有许多方法能帮你创建，删除和更改目录。
- mkdir()方法
    - 可以使用os模块的mkdir()方法在当前目录下创建新的目录们。你需要提供一个包含了要创建的目录名称的参数。
- 语法：`os.mkdir("newdir")`
- 例子：
- 下例将在当前目录下创建一个新目录test。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os
 
# 创建目录test
os.mkdir("test")
```

- chdir()方法
    - 可以用chdir()方法来改变当前的目录。chdir()方法需要的一个参数是你想设成当前目录的目录名称。
- 语法：`os.chdir("newdir")`
- 例子：
    - 下例将进入"/home/newdir"目录。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os
 
# 将当前目录改为"/home/newdir"
os.chdir("/home/newdir")
```

- getcwd()方法：
    - getcwd()方法显示当前的工作目录。
- 语法：`os.getcwd()`
- 例子：
    - 下例给出当前目录：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os
 
# 给出当前的目录
os.getcwd()
```

- rmdir()方法
    - rmdir()方法删除目录，目录名称以参数传递。
在删除这个目录之前，它的所有内容应该先被清除。
- 语法：`os.rmdir('dirname')`
- 例子：
    - 以下是删除" /tmp/test"目录的例子。目录的完全合规的名称必须被给出，否则会在当前目录下搜索该目录。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os
 
# 删除”/tmp/test”目录
os.rmdir( "/tmp/test"  )
```

### 文件、目录相关的方法
- 三个重要的方法来源能对Windows和Unix操作系统上的文件及目录进行一个广泛且实用的处理及操控，如下：
- File 对象方法: file对象提供了操作文件的一系列方法。
- OS 对象方法: 提供了处理文件及目录的一系列方法。

## Python File(文件) 方法

|   序号	|    方法及描述   |               |
|:-------------:|:-------------|:-----------------|
00001	|file.close()|关闭文件。关闭后文件不能再进行读写操作。
2	|file.flush()|刷新文件内部缓冲，直接把内部缓冲区的数据立刻写入文件, 而不是被动的等待输出缓冲区写入。
3	|file.fileno()|返回一个整型的文件描述符(file descriptor FD 整型), 可以用在如os模块的read方法等一些底层操作上。
4	|file.isatty()|如果文件连接到一个终端设备返回 True，否则返回 False。
5	|file.next()|返回文件下一行。
6	|file.read([size])|从文件读取指定的字节数，如果未给定或为负则读取所有。
7	|file.readline([size])|读取整行，包括 "\n" 字符。
8	|file.readlines([sizehint])|读取所有行并返回列表，若给定sizeint>0，返回总和大约为sizeint字节的行, 实际读取值可能比sizhint较大, 因为需要填充缓冲区。
9	|file.seek(offset[, whence])|设置文件当前位置
10	|file.tell()|返回文件当前位置。
11	|file.truncate([size])|截取文件，截取的字节通过size指定，默认为当前文件位置。
12	|file.write(str)|将字符串写入文件，没有返回值。
13	|file.writelines(sequence)|向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符。


## Python 异常处理
- python提供了两个非常重要的功能来处理python程序在运行中出现的异常和错误。你可以使用该功能来调试python程序。
    - 异常处理: 本站Python教程会具体介绍。
    - 断言(Assertions):本站Python教程会具体介绍。

- python标准异常

    |   异常名称	|      描述      |
    |------------|---------------|
    BaseException|	所有异常的基类
    SystemExit	|解释器请求退出
    KeyboardInterrupt	|用户中断执行(通常是输入^C)
    Exception	|常规错误的基类
    StopIteration	|迭代器没有更多的值
    GeneratorExit	|生成器(generator)发生异常来通知退出
    StandardError	|所有的内建标准异常的基类
    ArithmeticError	|所有数值计算错误的基类
    FloatingPointError	|浮点计算错误
    OverflowError|	数值运算超出最大限制
    ZeroDivisionError	|除(或取模)零 (所有数据类型)
    AssertionError	|断言语句失败
    AttributeError	|对象没有这个属性
    EOFError	|没有内建输入,到达EOF 标记
    EnvironmentError	|操作系统错误的基类
    IOError	|输入/输出操作失败
    OSError	|操作系统错误
    WindowsError	|系统调用失败
    ImportError	|导入模块/对象失败
    LookupError	|无效数据查询的基类
    IndexError	|序列中没有此索引(index)
    KeyError	|映射中没有这个键
    MemoryError	|内存溢出错误(对于Python 解释器不是致命的)
    NameError	|未声明/初始化对象 (没有属性)
    UnboundLocalError	|访问未初始化的本地变量
    ReferenceError	|弱引用(Weak reference)试图访问已经垃圾回收了的对象
    RuntimeError	|一般的运行时错误
    NotImplementedError	|尚未实现的方法
    SyntaxError	|Python 语法错误
    IndentationError	|缩进错误
    TabError	|Tab 和空格混用
    SystemError	|一般的解释器系统错误
    TypeError	|对类型无效的操作
    ValueError	|传入无效的参数
    UnicodeError	|Unicode 相关的错误
    UnicodeDecodeError	|Unicode 解码时的错误
    UnicodeEncodeError	|Unicode 编码时错误
    UnicodeTranslateError	|Unicode 转换时错误
    Warning	|警告的基类
    DeprecationWarning	|关于被弃用的特征的警告
    FutureWarning	|关于构造将来语义会有改变的警告
    OverflowWarning	|旧的关于自动提升为长整型(long)的警告
    PendingDeprecationWarning	|关于特性将会被废弃的警告
    RuntimeWarning	|可疑的运行时行为(runtime behavior)的警告
    SyntaxWarning	|可疑的语法的警告
    UserWarning	|用户代码生成的警告


### 异常处理
- 捕捉异常可以使用try/except语句。
- try/except语句用来检测try语句块中的错误，从而让except语句捕获异常信息并处理。
- 如果你不想在异常发生时结束你的程序，只需在try里捕获它。
- 语法：
    - 以下为简单的try....except...else的语法：

```python
try:
<语句>        #运行别的代码
except <名字>：
<语句>        #如果在try部份引发了'name'异常
except <名字>，<数据>:
<语句>        #如果引发了'name'异常，获得附加的数据
else:
<语句>        #如果没有异常发生
```

- try的工作原理是，当开始一个try语句后，python就在当前程序的上下文中作标记，这样当异常出现时就可以回到这里，try子句先执行，接下来会发生什么依赖于执行时是否出现异常。
- 如果当try后的语句执行时发生异常，python就跳回到try并执行第一个匹配该异常的except子句，异常处理完毕，控制流就通过整个try语句（除非在处理异常时又引发新的异常）。
- 如果在try后的语句里发生了异常，却没有匹配的except子句，异常将被递交到上层的try，或者到程序的最上层（这样将结束程序，并打印缺省的出错信息）。
- 如果在try子句执行时没有发生异常，python将执行else语句后的语句（如果有else的话），然后控制流通过整个try语句。


```python
try:
	fi = open("fojfdf","w")
	fi.write("ahfjahfjdfhsdkjal");
except IOError:
	print "can't find the file"
else:
	print "open file succeed"
finally:
	print "最后执行，成功或失败都会执行"
	
	try:
	fi = open("fojfdf","r")
	fi.write("write"); # 写入一个只读文件，发生异常
except IOError:
	print "can't find the file"
else:
	print "open file succeed"
finally:
	print "最后执行，成功或失败都会执行"
```


### 触发异常
- 我们可以使用raise语句自己触发异常pa
    - raise语法格式如下：
    - `raise [Exception [, args [, traceback]]]`
- 语句中Exception是异常的类型（例如，NameError）参数是一个异常参数值。该参数是可选的，如果不提供，异常的参数是"None"。
- 最后一个参数是可选的（在实践中很少使用），如果存在，是跟踪异常对象。

## Python类

### 类格式

```python
class ClassName:
    '帮助信息' # 类描述信息
    class_suite # 类内容
```

```python
class Student:
	"""docstring for Student"""
	score = 0.0 # 类变量
	# __init__ 类的构造函数或初始化方法
	def __init__(self,name,englishScore):
		self.englishScore = englishScore
		self.name = name
		Student.score += englishScore
	# 定义对象方法
	def displayScore(self):
		print "score:", Student.score	
	def displayStudent(self):
		print "name:",self.name,"Score:",self.englishScore

stu1 = Student("song",99.3)
stu2 = Student("zhang",22)
stu1.displayStudent()
stu1.displayScore()
stu2.displayStudent()
stu2.displayScore()
print "---------------"
stu1.englishScore = 120
stu1.displayStudent()
print "---------------"
# 访问对象属性的几个方法
print hasattr(stu1,"name")
print getattr(stu1,"name")
print setattr(stu1,"name","wang")
stu1.displayStudent()
print delattr(stu1,"name")
```

- 也可以使用以下函数的方式来访问属性：
    - getattr(obj, name[, default]) : 访问对象的属性。
    - hasattr(obj,name) : 检查是否存在一个属性。
    - setattr(obj,name,value) : 设置一个属性。如果属性不存在，会创建一个新属性。
    - delattr(obj, name) : 删除属性。
    
### Python内置类属性

- __dict__ : 类的属性（包含一个字典，由类的数据属性组成）
- __doc__ :类的文档字符串
- __name__: 类名
- __module__: 类定义所在的模块（类的全名是'__main__.className'，如果类位于一个导入模块mymod中，那么className.__module__ 等于 mymod）
- __bases__ : 类的所有父类构成元素（包含了以个由所有父类组成的元组）
- Python内置类属性调用实例如下：

```python
print "__doc__" ,Student.__doc__
print "__name__" ,Student.__name__
print "__module__",Student.__module__
print "__bases__",Student.__bases__
print "__dict__",Student.__dict__
```

### python对象销毁(垃圾回收)
- 同Java语言一样，Python使用了引用计数这一简单技术来追踪内存中的对象。
- 在Python内部记录着所有使用中的对象各有多少引用。
- 一个内部跟踪变量，称为一个引用计数器。
- 当对象被创建时， 就创建了一个引用计数， 当这个对象不再需要时， 也就是说， 这个对象的引用计数变为0 时， 它被垃圾回收。但是回收不是"立即"的， 由解释器在适当的时机，将垃圾对象占用的内存空间回收。
- 垃圾回收机制不仅针对引用计数为0的对象，同样也可以处理循环引用的情况。循环引用指的是，两个对象相互引用，但是没有其他变量引用他们。这种情况下，仅使用引用计数是不够的。Python 的垃圾收集器实际上是一个引用计数器和一个循环垃圾收集器。作为引用计数的补充， 垃圾收集器也会留心被分配的总量很大（及未通过引用计数销毁的那些）的对象。 在这种情况下， 解释器会暂停下来， 试图清理所有未引用的循环。

### 类的继承
- 面向对象的编程带来的主要好处之一是代码的重用，实现这种重用的方法之一是通过继承机制。继承完全可以理解成类之间的类型和子类型关系。
- 需要注意的地方：继承语法 class 派生类名（基类名）：//... 基类名写作括号里，基本类是在类定义的时候，在元组之中指明的。
- 在python中继承中的一些特点：
    - 1：在继承中基类的构造（__init__()方法）不会被自动调用，它需要在其派生类的构造中亲自专门调用。
    - 2：在调用基类的方法时，需要加上基类的类名前缀，且需要带上self参数变量。区别于在类中调用普通函数时并不需要带上self参数
    - 3：Python总是首先查找对应类型的方法，如果它不能在派生类中找到对应的方法，它才开始到基类中逐个查找。（先在本类中查找调用的方法，找不到才去基类中找）。
- 如果在继承元组中列了一个以上的类，那么它就被称作"多重继承" 。

- 可以使用issubclass()或者isinstance()方法来检测。
    - issubclass() - 布尔函数判断一个类是另一个类的子类或者子孙类，语法：issubclass(sub,sup)
    - isinstance(obj, Class) 布尔函数如果obj是Class类的实例对象或者是一个Class子类的实例对象则返回true。

- 方法重写
    - 如果你的父类方法的功能不能满足你的需求，
- 运算符重载
    - Python同样支持运算符重载
### 类属性与方法
- 类的私有属性
    - __private_attrs：两个下划线开头，声明该属性为私有，不能在类地外部被使用或直接访问。在类内部的方法中使用时 self.__private_attrs。
- 类的方法
    - 在类地内部，使用def关键字可以为类定义一个方法，与一般函数定义不同，类方法必须包含参数self,且为第一个参数
- 类的私有方法
    - __private_method：两个下划线开头，声明该方法为私有方法，不能在类地外部调用。在类的内部调用 self.__private_methods
