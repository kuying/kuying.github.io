---
layout: post
title: css3之三:字体和文本, 布局
description: "css学习笔记之三, 字体和文本, 布局"
modified: 2013-10-04
tags: [web]
image:
  feature: texture-feature-04.jpg
comments: true
share: true
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

### 字体

#### 来源

    用户机器中安装的字体
    保存在第三方网站上的字体, 最常见的是Typekit和Google, 可以使用link标签将之链接到页面上
    保存在web服务器上的字体, 可以使用@font-face规则和网页一起发送给浏览器

##### 公共字体库

Google Web Fonts 和 Adobe的Typekit 两个最大的在线公共字体库, 订购方式提供访问

##### 打包的@font-face包

会在页面第一次加载时被浏览器下载并缓存

[Font Squirrel](http://www.fontsquirrel.com)提供很多现成的字体包

#### 字体族 font-family

设定元素中的文本使用什么字体, 一般来讲应该给整个页面设定一个主字体(设定body元素), 对特殊元素再应用font-family

指定文本字体, 需要多列出几种后背字体, 以防第一种字体无效, 这个字体列表即字体栈, 对于已包含空格的字体名(多余一个单词)还要加上引号, 一般给字体栈的最后一项指定一个通用字体

通用字体

- serif 衬线字体, 每个字符笔画末端有一些装饰线
- sans-serif 无衬线字体
- monospace 等宽字体, 又叫代码体
- cursive 草书或手写体
- fantasy 不能归入其他类别的字体, 一般是奇形怪状的字体

#### 字体大小 font-size

使用绝对单位, 如像素或点; 或使用相对单位, 如百分比或em或rem(根元素的字体大小)

font-size可被继承, 如果改变一个元素的字体大小, 可能会导致其子元素大小成比例的变化; 使用绝对单位设定的元素不会继承祖先元素的字体大小

#### 字体样式 font-style

italic/oblique normal

#### 字体粗细 font-weight

100, 200, ... 900 或 lighter normal bold bolder

最好只使用 normal和bold

#### 字体变化 font-variant

small-caps, normal

small-caps 所有小写英文字母变成小型的大写字母

经常将small-caps用于 ::first-line伪元素

#### 简写属性 font

- 必须声明 font-size和font-family的值
- 必须按照如下顺序声明
    1. font-weight, font-style, font-variant部分先后
    2. 然后时font-size
    3. 最后时font-weight

### 文本

#### 文本缩进 text-indent

设定行内盒子相对于包含元素的起点, 默认是包含元素的左上角, 实现首行缩进

可被继承

#### 字符间距 letter-spacing

一定要使用相对单位, 以便字间距能随字体大小同比例变化

#### 单词间距 word-spacing

调整单词间距

#### 文本装饰 text-decoration

underline(下划线), overline(上划线), line-through(上下划线), blink(闪烁), none

#### 文本对齐 text-align

left, right, center, justify(两端对齐)

#### 行高 line-height

行高平均分布在一行文本的上方和下方

font快捷属性支持符合形式的 font-size和line-height

{% highlight css %}
div#intro {font:1.2em/1.4 helvetica, arial, sans-serif;}
{% endhighlight %}

#### 文本转换 text-transform

转换元素中文本的大小写

none uppercase(全大写) lowercase(全小写) capitalize(首字母大写)

#### 垂直对齐 vertical-align

值: 任意长度或sub super top middle bottom

这个属性只影响行内元素


### 布局

- 固定宽度布局

    不会随用户调整浏览器大小而变化, 一般是900到1100像素, 以960最常见

- 流动布局

    随用户调整浏览器窗口大小而变化

- 弹性布局

    与流动布局类似, 浏览器变宽时不仅布局变宽而且所有内容元素大小也会变化, 太过复杂而难以设计   
    
使用CSS媒体查询可以让基于浏览器窗口宽度而提供不同的CSS样式, 使用各种屏幕宽度的可变固定布局即响应式布局, 取代流动布局

- 高度不必设定, 保持元素的height属性为auto
- 精细的控制宽度, 浏览器窗口宽度变化, 布局能适当调整

#### 三栏-固定宽度布局

#### 三栏-中栏流动布局

#### 多行多栏布局