---
layout: post
title: HTML的几个重要概念
description: HTML学习笔记
keywords: HTML,HTML元素,HTML标签,HTML属性,Web前端
category: Web前端
---

## HTML

+ HTML是用来制作网页的标记语言
+ HTML是Hypertext Markup Language的英文缩写,即超文本标记语言
+ HTML语言是一种标记语言,不需要编译,直接由浏览器执行
+ HTML文件是一个文本文件,包含了一些HTML元素,标签等.HTML文件必须使用html或htm为文件名后缀
+ HTML是大小写不敏感的,HTML与html是一样的
+ HTML是由W3C的维护的

### HTML的历史

- HTML 1.0 -- 1993年6月,IETF发布
- HTML 2.0 -- 1995年11月,发布
- HTML 3.2 -- 1996年1月,W3C推荐标准
- HTML 4.0 -- 1997年12月,W3C推荐标准
- HTML 4.01 -- 1999年12月,W3C推荐标准
- HTML 5.0 -- 2008年8月,W3C工作草案

## HTML标签

HTML标签是HTML语言中最基本的单位,HTML标签是HTML语言最重要的组成部分.

+ 通常要用两个角括号括起来:<和>.
+ 都是闭合的(闭合就是标签的最后要有一个/,来标示结束.),但不一定是成对出现的,比如<body>和</body>一对标签.(<body>是开始标签,</body>是结束标签,在开始和结束标签中可以有内容),比如<br />就是单独的.(注意要在最后加上/,以标示其是独立的)
+ 标签是大小写无关的,<body>跟<BODY>表示的意思是一样的.标准推荐使用小写.

### HTML标签语法

HTML标签(两种形式,成对与不成对):

{% highlight html %}
	<标签名>内容</标签名>
	<标签名 />
{% endhighlight %}

例如：

{% highlight html %}
	<table> </table>
{% endhighlight %}

表格的开始标签与结束标签

{% highlight html %}
	<br />
{% endhighlight %}

单独出现的换行标签

## HTML属性

一般都出现在HTML标签中,HTML属性是HTML标签的一部分.

+ 标签可以有属性,它包含了额外的信息.属性的值一定要在双引号中
+ 标签可以拥有多个属性
+ 属性由属性名和值成对出现

### HTML属性语法

{% highlight html %}
	<标签名 属性名1="属性值" 属性名2="属性值" ... 属性名N="属性值"></标签名>
{% endhighlight %}

例如：

{% highlight html %}
	<table summary="html table" border="0">
{% endhighlight %}

标签<table>是表格标签.使用border属性,定义没有边框的表格.使用summary属性定义表格的简短描述.

## HTML元素
 
HTML元素是构建网页的一种单位,是由HTML标签和HTML属性组成的,HTML元素也是网页中的一种基本单位.

例如：

{% highlight html %}
	<a href="http://ic3evil.github.com">
    冰神舞's Blog
    </a>
{% endhighlight %}

这个是一个链接元素：[冰神舞's Blog](http://ic3evil.github.com)



