---
title: html入门
date: 2022-02-21 13:34:29
tags: html
categories: ProgLang
---

<div></div>

<!--more-->



# html 入门

这学期的可视化课程第一次作业需要使用html，这篇blog描述的是我在w3school.com.cn上学习html的过程。

[toc]

## tags

* \<html>定义HTML文档
* \<body>定义文档主体



* h1 - h6 --> 标题等级

```html
<h1>一级标题</h1>
```



* p --> paragraph

```html
<p> </p>
```



* q --> 短引用, quote
  * \<blockquote cite="a url">长引用

```html
<q>quotation</q>
```



* abbr --> 定义缩写

```html
<abbr title="world health organization">WHO</abbr>
```



* address --> 联系信息

```html
<address>
Written by Donald Duck.<br> 
Visit us at:<br>
Example.com<br>
Box 564, Disneyland<br>
USA
</address>
```



* cite --> 著作标题

```html
<p><cite>The Scream</cite> by Edward Munch. Painted in 1893.</p>
```



* 双向重写 --> bdo (bi-directional overrite)

```html
<bdo dir="rtl">This text will be written from right to left</bdo>
```



* a 定义链接
  * 在href属性中指定链接
  * target="_blank"在新窗口打开

```html
<a href="jerryji.site">This is a link</a>
```



* img 定义图像
  * alt属性是当图像无法加载时可替换的文本

```html
<img src="w3school.jpg" width="104" height="142" />
```



* 空元素 - 没有内容的HTML元素， 在开始标签中关闭
  * \<br />用于换行
  * \<hr />用于创建水平线s
  * \<!-- This is a comment -->

```html
<br>或者<br />
```

## 属性

形式 -- name="value"，在开始标签中设定

比如上面链接中的href属性

### 更多属性

1. \<h1 align="center">
2. \<body bgcolor="yellow">
   * 颜色名字有16种（否则hex)
     * **aqua、black、blue、fuchsia、gray、green、lime、maroon、navy、olive、purple、red、silver、teal、white、yellow**
3. \<table border="1">

如果属性值包含双引号，则必须使用单引号



### style属性

用于改变HTML元素的样式，淘汰了很多样式

* style="background-color:red"

* style="font-family:arial;color:red:font-size:20px;"
* style="text-align:center"



## 样式表

* 外部

  ```html
  <head>
  <link rel="stylesheet" type="text/css" href="mystyle.css">
  </head>
  ```

* 内部

  ```html
  <head>
  
  <style type="text/css">
  body {background-color: red}
  p {margin-left: 20px}
  </style>
    
  </head>
  ```

* 内联

  ```html
  <p style="color: red; margin-left: 20px">
  This is a paragraph
  </p>
  ```



## 表格

```html
<table border="1">
<tr>
  <th>姓名</th>
  <th>电话</th>
  <th>电话</th>
</tr>
<tr>
  <td>Bill Gates</td>
  <td>555 77 854</td>
  <td>555 77 855</td>
</tr>
</table>
```

\<tr>定义每一行

\<td>定义每个单元格的内容

* 如果单元格是空的，用\&nbsp;占位符以显示边框



## 列表

* 无序列表 unordered list

```html
<ul>
<li>Coffee</li>
<li>Milk</li>
</ul>
```

* 有序列表为\<ol>
* 自定义列表\<dl>
  * 每一项用\<di>开始



## div和span

### HTML 块元素与内联元素

大多数 HTML 元素被定义为块级元素或内联元素。



块级元素在浏览器显示时，通常会以新行来开始（和结束）。

例子：`<h1>, <p>, <ul>, <table>`



内联元素在显示时通常不会以新行开始。

例子：`<b>, <td>, <a>, <img>`



### div

div是块级元素，没有特定含义，用于排版，设置属性等

### span

span是内联元素，作为文本的容器，可以为文本设置属性



## HTML 类

```html
<!DOCTYPE html>
<html>
<head>
<style>
.cities {
    background-color:black;
    color:white;
    margin:20px;
    padding:20px;
} 
</style>
</head>

<body>

<div class="cities">
<h2>London</h2>
<p>
London is the capital city of England. 
It is the most populous city in the United Kingdom, 
with a metropolitan area of over 13 million inhabitants.
</p>
</div> 

</body>
</html>
```



```html
<!DOCTYPE html>
<html>
<head>
<style>
  span.red {color:red;}
</style>
</head>
<body>

<h1>My <span class="red">Important</span> Heading</h1>

</body>
</html>
```



