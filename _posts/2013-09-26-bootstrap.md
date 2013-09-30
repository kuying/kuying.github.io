---
layout: post
title: bootstrap学习
description: "bootstrap"
modified: 2013-09-26
tags: [web]
image:
  feature: texture-feature-01.jpg
comments: true
share: true
---

[bootstrap 学习](http://getbootstrap.com/)

[github](https://github.com/twbs/bootstrap)

[bootstrap 中文](http://www.bootcss.com/)

[中文v2.0.2](http://wrongwaycn.github.io/bootstrap/docs/)

---

twitter web用户界面和交互架构, js部分依赖jquery.js

如果不是IE9浏览器, 模拟实现html5
{% highlight html %}
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<!--[if IE 9]>
    <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
<![endif]-->
{% endhighlight %}

\<div\>容器标签

### 内置响应式网格系统

网格布局, 将网页分为12栏, 使用span + 栏数 (span4, span8)

container 设置宽度 并居中显示

row 一排显示

span? 占用网格宽度

嵌套布局, 子布局的宽度栏数和应该小于或等于父布局的宽度栏数

#### 流动布局

container-fluid 占用百分比的宽度

row-fluid 占用百分比的宽度

嵌套布局下, 子布局的宽度和应该等于12

#### 响应浏览器设置布局

根据CSS3 media queries

@media (max-width: 767px) and (min-width: 579px) {} 表示浏览器宽度在579和767之间时, 应用大括号中的样式

#### bootstrap-responsive.css

根据设备宽度设置布局 span的宽度

.visible-phone .hidden-phone

.visible-table .hidden-table

.visible-desktop .hidden-desktop

是否显示元素, 使用display属性

### 常用文字排版标签及样式

\<p\> 段落

\<strong\> 加粗

\<em\> 斜体

\<abbr title="xxxxx" \> 简称标识

\<blockquote \> 文字引用 (灰色左边框)

\<small\> 简称标识

\<cite\> 网站

.initialian 大写

.pull-right 右对齐

### 列表

无序列表

\<ul\>

\<li\> 列表没一项

有序列表

\<ol\>

\<li\>

描述列表

\<dl\> 

\<dt\>内容 \<dd\> 解释部分

更改描述列表的样式 \<dl class="dl-horizontal"\>

### 代码内容

显示< > => &gt; &lt;

\<code\> 行内的代码

\<pre\> 格式化的代码块

google-prettify 显示代码更漂亮

### 表格

结构

{% highlight html %}
<table>
	<thead>
		<tr>
			<th> ... </th>
			<th> ... </th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td> ... </td>
			<td> ... </td>
		</tr>
	</tbody>
</table>
{% endhighlight %}

在table标签上添加class="table"	

	table
	table-striped 条纹表格 奇偶数排使用不同的颜色
	table-bordered 带边框
	table-condensed 缩小表格内边距

### 表单

input标签种类 http://www.w3.org/wiki/HTML/Elements/input 

#### 文本框

\<input type="text" placeholder="提示信息" \>

使用class="span?" 设置宽度

帮助文字格式

	help-inline
	help-block

文本框的前缀 后缀
{% highlight html %}
<div class="input-prepend inpupt-appedn">
<span class="add-on">&yen;</span><input type="text"><span class="add-on">.00</span>
</div>
{% endhighlight %}


#### 单选按钮

\<label class="radio"\>

\<input type="radio" name="???", value="male"\>男\</label\>

\<label class="radio"\>

\<input type="radio" name="???", value="female"\>女\</label\>

#### 多选

\<label class="checkbox"\>

\<input type="checkbox" name="???", value="female"\>女\</label\>

#### 单选/多选列表

{% highlight html %}
<select>
	<option value="???">????</option>
</select>
{% endhighlight %}

修改select的属性, 设置multiple="multiple"

#### 表单排版

	form-horzontal
	control-group
	controls
	control-label
	
	form-action

### 按钮

	btn
	btn-primary
	btn-info
	btn-success
	btn-warning
	btn-danger
	btn-inverse
 
	btn-large
	btn-small
	btn-mini

小图标

使用\<i class="icon-xxxx"\>标签

icon图标种类 http://getbootstrap.com/2.3.2/base-css.html#icons

icon颜色 icon-white ...

#### 按钮群组

	btn-toolbar
	btn-group

#### 下拉菜单按钮

jquery + bootstrap的dropdown

### 导航菜单

	navbar
	navbar-inner

#### 导航菜单的下拉

dropdown

