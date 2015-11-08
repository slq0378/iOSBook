# html总结

- 简介

```html
<!DOCTYPE html>
<html>
<body>

<h1>我的第一个标题</h1>
	<h2>我的第二个标题</h2>

<p>我的第一个段落。</p>

</body>
</html>
```

- 注意：DOCTYPE 声明了文档类型

## HTML 标题
- h1 - h6 来表示标题的大小

```html
<!DOCTYPE html>
<html>
<body>

<h1>这是标题 1</h1>
<h2>这是标题 2</h2>
<h3>这是标题 3</h3>
<h4>这是标题 4</h4>
<h5>这是标题 5</h5>
<h6>这是标题 6</h6>

</body>
</html>
```

## 段落
- p 标签会自动换行

```html
<!DOCTYPE html>
<html>
<body>

<p>这是一个段落。</p>
<p>这是一个段落。</p>
<p>这是一个段落。自动换行</p>

</body>
</html>
```

## 链接
- a 标签指定一个链接，使用如下指定`href`

```html
<html>
<body>

<a href="http://www.w3cschool.cc">这是一个链接</a>

</body>
</html>
```
## 图像
- img标签指定一个要显示的图片，指定src和宽高就可以显示出来

```html
<html>
<body>

<img src="/images/w3cschool.png" width="300" height="120" />

</body>
</html>
```

## html元素
- 每一个html中都会有若干个html元素组成。
- 比如以上图像标签的显示代码中，包含三个html元素，p标签，body标签，html标签。
- 而且这些标签都是成对出现的，有其实标签和结束标签。

## HTML 属性
- 属性是 HTML 元素提供的附加信息。
    - HTML 元素可以设置属性
    - 属性可以在元素中添加附加信息 about an element
    - 属性一般描述于开始标签
    - 属性总是以名称/值对的形式出现，比如：name="value"。
    - 属性值应该始终被包括在引号内,双引号是最常用的，不过使用单引号也没有问题。