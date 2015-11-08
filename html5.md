# HTML5

## 简介

- HTML5的使用
	- 1> 自己编写大量的HTML5代码
	- 2> 使用现成的HTML5框架
		- **sencha-touch**
		- **phoneGap**
		- **jQuery mobile**
		- **bootstrap**

- 手机APP的开发模式
	- 1> 原生（纯OC）
	- 2> 纯HTML5
	- 3> 原生+HTML5

- 开发工具
	- **WebStrom**

- 网页组成
	- 1、具体内容
	- 2、网页样式 **CSS**
	- 3、交互事件 **JavaScript**
- 学习网站
	- <http://www.w3school.com.cn/>

## 语法
- 常见的HTML标签

```
	标题：h1、h2、h3、h4、h5....
	段落：p
	换行：br
	容器：div、span（用来容纳其他标签）
	表格：table、tr、td
	列表：ul、ol、li
	图片：img
	表单：input
	链接：a
```

### HTML5新增标签

- HTML5新增了27个标签元素,废弃了16个标签元素,主要包括结构性标签、级块性标签、行内语义性标签、交互性标签。

- 1.结构性标签
	- 负责Web上下文结构的定义，确保HTML文档，包括：
	- **article**  文章主体内容(一篇博客、一篇论坛帖子、一段用户评论、插件)
	- **header**   标记头部区域内容
	- **footer**   标记脚部区域内容
	- **section**  区域章节表述
	- **nav**      菜单导航,链接导航
- 2、块级性标签
	- 完成Web页面区域的划分，确保内同的有限分割，包括：
	- **aside** 	注记，贴士，侧栏，摘要，插入的引用作为补充主体的内容
	- **figure** 	对过个元素组合并展示的元素，常与**figcaption**联合使用
	- **code**		表示一端代码块
	- **dialog** 	人与人之间对话，包含dt和dd两个组合元素（dt用于表示说话者，dd用于表示说话者的内容）
- 3、行内语义性标签
	- 完成Web页面具体内容的引用和表述，丰富展示内容，包括：
	- **meter** 		特定范围内的数值，如工资、数量、百分比
	- **time**			时间值
	- **progress**	进度条，可用max、min、step进行控制器，完成对进度的表示和监听
	- **video**		视频元素，用于视频播放，支持缓冲预载和多种视频媒体格式
	- **audio**		音频元素，用于音频播放，支持缓冲预载和多种音频媒体格式
- 4、交互性标签
	- 功能性内容的表达，有一定的内容和数据的关联，是各种事件的基础，包括：
	- **details**		表示一段具体的内容，默认不显示，通过某种方式（单击）与**legend**交互才会显示
	- **datagrid** 	控制客户端与数据显示，可用于动态脚本及时更新
	- **menu**			用于交互菜单
	- **command**		用于处理命令按钮
### CSS
- **CSS**的全称是**Cascading Style Sheets**，层叠样式表.
	- CSS的编写格式是键值对形式
	- 如color: red;冒号:左边的是属性名，冒号:右边的属性值
- CSS三种使用方式
	- 行内样式：（内联样式）直接在标签的style属性中书写
		- <body style="color: red;">

	- 页内样式：在本网页的style标签中书写
		```
				<style>
				    body {
				        color: red;
				    }
				</style>
		```
	- 外部样式：在单独的CSS文件中书写，然后在网页中用link标签引用
		- <link rel="stylesheet" href="index.css">

- CSS原则
	- css遵循一个规律:
		- 1.就近原则
		- 2.叠加原则
- CSS选择器原则
	- css选择器遵循:
		- 1. 在相同级别:1.叠加原则  2.就近原则
		- 2. id选择器> 类选择器 > 标签选择器
		- 3. 范围越小,优先级别越高
### 选择器

- 标签选择器 `div{color:blue;}` 根据标签名来写，统一用于div标签
- 类选择器	`.one{color:red;}`
- id选择器 `#one{color:red}` 只能用于一个标签，唯一
- 并列选择器 `div,.high{color:red;}` 相当于同时作用于div标签和类high
- 符合选择器 `div.high{color:red;}` 作用于div标签中遵守class的标签
- 后代选择器 `div p{color:red;}` 作用于div中的p标签
- 直接后代选择器 `div > p {color:red;}` 只作用于儿子
- 相邻兄弟选择器 `div + p {color:red;}`只作用于兄弟
- 属性选择器 `div[name]{color:red;}` `div[name][age]{color:red;}` `div[name="jack"]{color:red;}`直接作用于具体标签
- 伪类
	- :active
	- :focus
	- :hover



- 选择器优先级

```
	选择器的针对性越强，它的优先级就越高
		选择器的权值
		通配选择符（*）：0
		标签： 1
		类： 10
		属性： 10
		伪类： 10
		伪元素： 1
		id： 100
		important： 1000
		原则：选择器的权值加到一起，大的优先；如果权值相同，后定义的优先
```

- 都是隐藏内容,区别:
	- 1.display会连同尺寸一起隐藏
	- 2.visibility不会隐藏尺寸
	- display: none;
	- visibility: hidden;
## CSS布局
- 标准流
	- 从上到下，从左到右
- 脱离标准流
	- float属性（left、right）
	- position属性（top、right、bottom、left）


# JavaScript
- JS浏览器客户端脚本语言（NetSpace公司与Sun）
## Node.js
- Node是JS运行环境，是对Google V8引擎的封装
- Node优势
	- 可做后台语言
	- 单线程：（事件轮训：）
- JS书写方式
	- 页内
```
	<script type='text/javascript'>
		alert('hello world');
		console.log('hahhaha');
	</script>
```

	- 外部
	<script scr=""> </script>
- 语法规则
	- 变量定义 （var num = 10;）

	```
	// 定义变量
	var number = 10;
	var money = 100.99;
	var name = 'jack';
	var name2 = 'rose';
	var result = true; // false
	// 输出
	console.log(numer,money,name,result);
	console.log(typeof numer,typeof money,typeof name,typeof name, typeof result);
	// 类型:数值型都是number，字符串是string，布尔类型是boolan
	// 拼接
	var names = name + name2;
	// 运算：从左到右，任何类型遇到string都会被转换成string
	var str1 = 10 + 10 + '10'; // 20 10
	var str2 = '10' + 10 + 10; // 10 20
	var str3 = 10 + '10' + 10; // 10 10 10
	// 数组 object对象
	var numbers = [number,12,'lala',result,[1000,'haha']];
	consol.log(numbers); // 输出所有
	consol.log(numbers[5]); // 根据下标输出
	```

	- 函数 `function`

	```
	// 注意:没有返回值，参数列表不用指定类型
	function sum(num1,num2) //
	{
		return num1 + num2;
	}
	// 传入数组
	function sum(numbers)
	{
		var count = 0;
		for(var i = 0; i < numbers.length; i ++)
		{
			count += numbers[i];
		}
		return count;
	}
	// 升级版本,arguments是内部含有数组
	function sum()
	{
		var count = 0;
		for(var i = 0; i < arguments.length; i ++)
		{
			count += numbers[i];
		}
		return count;
	}
	// 调用
	var result = sum(10,20,30);
	//
	// 匿名函数
	var result = function(){
		console.log('-----');
	}
	// 调用匿名函数
	result();
	```

	- 对象

	```
	var dog = {
		name : 'wangcai';
		height : 1.5
		age : 10
		firends : ['xiaohuang','xiaohua']
		// 定义功能、方法
		run : function(){
			console.log('遛弯去了');
		}
		eat : function(meat){
		// this 指针，指向当前对象
			console.log(this.name + '吃' + meat);
		}
	}
	// 输出
	for(var pname in dog)
	{
		console.log(pname,dog[pname]);
	}
	// 直接输入属性
	dog.age;
	// 调用方法
	dog.run();
	dog.eat('骨头');
	```

	- 构造函数 ： new
	- 通过new生成多个对象

	```
	var Dog = function(){
		this.name = null;
		this.height = null;
		this.age = null;
		this.run = function(){
			console.log('遛弯');
		}
		this.eat = function(meat){
			console.log(this.name + '吃' + meat);
		}
	}
	// 使用
	var dog1 = new Dog();
	dog1.name = 'wangcai';
	var dog2 = new Dog();
	dog2.name = 'ahuang';
	dog1.run();
	dog2.eat('骨头');
	```

	- 构造函数：带参数

	```
	var Dog = function(name,height,age){
	this.name = age;
	this.height = height;
	this.age = age;
	this.run = function(){
		console.log('遛弯');
	}
	this.eat = function(meat){
		console.log(this.name + '吃' + meat);
	}
	var dog1 = new Dog('旺财',1.2,10);
	dog1.run();
	```

	- && 和 ||

	```
	var name1 = '';
	var name2 = 'name2';
	var name3 = 'name3';
	var name4 = 'name4';
	//
	var newName = name1 || name2 || name3 || name4;
	console.log(newName);
	var age = 22;
	(age > 20) && console.log('滚动吧');
	```

	- 内置对象 window document

	```
<script>
	// window
	// 1、所有全局变量都是他的属性
	// 2、所有全局函数都是他的函数
	var num = 10;
	function sum(){
		conslole.log('hahahah');
	}
	num;
	window.num;
	// 方法
	sum();
	window.sum();
	// document
	// 1、动态获取网页中所有节点
	// 2、可以动态的最节点进行增删改查（CRUD）
	//
	document.write('hello world');
	document.write('<p>hsdhfhdsfhakdjsfhadsjkfa</p>');
	function change(){
		// 这个id就是标签选择器指定的id
		// 这个className就是标签的类选择器
		// 这个name就是标签的name属性
		// 这个tagName就是按照标签类型寻找符合条件的标签
		var img1 = document.getElementById('icon');
		var img2 = document.getElementsByClassName('test2');
		var img3 = document.getElementsByName('test3');
		var img4 = document.getElementsByTagName('img');
		img.src = 'img/1/jpg';
		img.style.width = '100px';
	}
	// lastIndexOf(‘1.img’)// 获取符合条件的字符串，没有就返回-1
</script>
	```

	- 使用方法
	- 如果加载的外部js，那么再在这个脚本里添加其他操作，是不起作用，可以新建一个脚本标签
	```
<script>
	var img = document.getElementsByName('icon')[0];
	// 定时器: ms计算
	var timer = setInterval(function(){
		alert(img.src);
		if(div.innerHTML == 1)
		{
			clearInterval(timer);
		}
	},1000);
	// 给按钮添加事件,第一种是在标签里直接指定，或者在js脚本里指定
	div.onmouseup = function(){
		div.style.background = 'green';
	}
</script>
	```

	- CRUD

	```
	// 1、插入
	document.write('hello world');
	document.write('<img src = "img/img_01.jpg">');// 注意不要写成../img/img_01.jpg
	// 2、创建
	var btn = document.createElement('button');
	btn.innerHTML = '百度一下';
	btn.style.width = '100';
	// 把按钮加入到body里
	document.body.appendChild(btn);
	// 把按钮添加在子标签里
	var div = document.getElementsByClass('test')[0];
	div.appendChild(btn);
	// 3、删除
	var img = document.getElementsByClassName('icon')[0];
	// 方式1:找打
	img.parentNode.removeChild(img);
	// 方式2
	img.remove();
	```

	- 练习

	```
	// == 值比较
	// === 类型比较
	function $(id){
		return typeof id === document.getElementById()id;
	}
	```


## JQuery

- JS存在的问题
	- JS存在兼容性的问题
	- 代码相对复杂，较多
- JQuery
	- 能够轻松实现DOM操作
	- 很少得代码，写更多的功能
	- 实现各种特效和动画

- 代码

```
$('img').attr('src','img/img_01.jpg');
$('.test1').attr('width','200px');
$('img').clck(function(){
	$('img').attr('src','img/img_02.jpg');
});
// 动画
$('btn').click(function(){
	$('img').toggle('linear');// 线性动画
});
```

- HTML和OC通信

- HTML访问OC
- 在控制器里拦截请求（代理方法）

## WebView
- Xcode7下使用webView
![](/images/Xcode7加载webView需要设置plist.png)



