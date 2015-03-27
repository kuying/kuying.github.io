---
layout: post
title: javascript
description: "js"
modified: 2014-12-21
category: articles
tags: [web]
image:
  feature: texture-feature-05.jpg
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

    Javasctipt权威指南v6

### js核心

#### 词法

1.	注释
	* // 单行注释
	* /* */ 多行注释
2.	直接量 - 程序中直接使用的数据值
3.	标识符和保留字
	* 起始字符：字母、下划线、$
	* 后续字符：字母、下划线、$、数字

#### 类型、值和变量

| 原始类型 | 数字、字符串、布尔值、null、undefined |
| 对象类型 | 其他，对象是属性的集合，由key-value对构成 |

变量 无类型untyped

1.	数字
	*	不区分整数和浮点
	*	整形直接量
		*	十进制：数字序列
		*	十六进制：'0x'或'0X'前缀 + 十六进制数串
	*	浮点型直接量
		[digits][.digits][(E\e)[(+|-)]digits]
		*	传统实数写法：整数部分，小数点，小数部分
		*	指数计数法：实数 + e或E + 正负号 + 整形指数
	*	算术运算符 + - * / %
	*	Math对象
		* Math.pow(x, y)	// 幂
		*	Math.exp(x)			// 三次幂
		* Math.round(x)		// 四舍五入
		* Math.ceil(x) 		// 向上取整
		* Math.floor(x)		// 向下取整
		* Math.abs(x)			// 绝对值
		* Math.max(x, y, z)	// 最大值
		* Math.min(x, y, z)	// 最小值
		* Math.random()		// 大于0 小于1.0的伪随机数
		* Math.PI					// 圆周率 π
		* Math.E					// 自然对数的底数 e
		* Math.sqrt(x)		// 平方根
		* Math.sin(x)			// 三角函数
		* Math.cos(x)			// 三角函数
		* Math.atan(x)		// 三角函数
		* Math.log(x)			// 自然对数
	*	溢出overflow，下溢underflow，除零不会报错
		*	Infinity, NaN
		* NaN 和任何值都不相等，isNaN() isFinite()
	* 浮点数精度错误
		*	.3 - .2 != .1
	*	日期和时间 Date
		*	var now = new Date() // 当前时间和日期
		*	var then = new Date(2014, 12, 21) // 2014年12月21日
		*	var later = new Date(2014, 12, 21, 19, 51, 21) // 2014年12月21日, 19:51:21
		*	var elapsed = now - then // 时间间隔，毫秒
		*	later.getFullYear() // 年
		*	later.getMonth() // 月
		*	later.getDate() // 天数
		*	later.getDay() // 星期几
		*	later.getHours() // 当地时间，时
		*	later.getUTCHours() // UTC时间，时
2.	文本
	*	转义字符 \
		*	\n 换行			\u000A
		*	\v 垂直制表符	\u000B
		*	\f 换页符		\u000C
		*	\r 回车符		\u000D
		*	\\" 双引号		\u0022
		*	\\' 单引号		\u0027
		*	\\\\ 反斜线		\u005C
		*	\xXX 两位十六进制数XX制定的Latin-1字符
		*	\uXXXX 4位十六进制数XX制定的Unicode字符
	* 字符串方法，可当做只读数组
		* s1 + s2		// 字符串连接
		*	s.length	// 长度
		* s.charAt(n)	// 第n个字符
		* s.substring(x, y)	// x~y个字符
		* s.slice(x, y)	// 同上
		*	s.indexOf(c)	// 字符c首次出现的位置
		* s.lastIndexOf(c)	// 字符c最后一次出现的位置
		* s.split(x)		// 分割字符串
		* s.replace(x, y)	// 全文替换
		* s.toUpperCase()	// 大写转换
	*	匹配
		*	RegExp不是基本类型
		* 方法
			pattern = /\d+/g // 直接量写法
			pattern.test(s)
		* 字符串search match replace split 支持RegExp参数
3.	布尔
	*	undefined null 0 -0 NaN ""(空字符串) 转换为false，其他都被当做true
4.	null和undefined
5.	全局对象 global object
6.	包装对象 存取字符串、数字或布尔值的属性时临时创建的对象
	*	可以通过String() Number() Boolean()显示创建包装对象
7.	不可变原始值和可变的对象引用
	* 原始值的比较是值的比较
	*	对象的比较是引用的比较(当且仅当引用同一个基对象菜相等)
8.	类型转换
	* ==
	*	显式转换 Boolean() Number() String() Object() 不使用new运算符调用
	* 运算符隐式转换
		*	! 转换bool
		*	x + "" // 等价String(x)
		*	+x, x - 0 // 等价 Number(x)
		*	!!x // 等价Boolean(x)
	*	Number->String
		*	toString(n) 指定转换基数
		*	toFixed(n) 指定转换小数点后位数
		*	toExponential(n) 指定转换小数点后位数，小数点前只有一位，指数计数法
		* toPrecisioin(n) 指定有效位，有效位数小于数字整数部分位数则使用指数计数法
	* String->Number, 全局函数
		*	parseInt(str)
		*	parseFloat(str)
	* toString() 和 valueOf()
	* + == != 将对象转化为原始值(基本上是数字)，日期对象转化为字符串
	* < 等关系运算符 将对象转化为原始值，日期对象转化为数字
	* - 转化为数字
9.	变量
	*	声明 var
	*	作用域，只有函数作用域，在函数中声明的所有变量在函数体内始终可见
	*	作用域链

#### 表达式和运算符

1.	原始表达式，不能在包含其他表达式
	*	常亮/直接量
	*	关键字
	*	变量
2.	对象和数组的初始化表达式
	* [...]
	*	{...}
3.	函数定义表达式
4.	属性表达式
	*	expression . idengifier
	* expression [ expression ]
5.	调用表达式
	*	expression ( para, ...)
6. 对象创建表达式, 不传入任何构造参数可以省略括号
	*	new Object()
7.	运算符
	*	++	前/后增量
	*	\-\-	前/后减量
	*	\-	求反
	*	\+	转换为数字
	* ~	按位求反
	* !	逻辑非
	*	delete	删除属性
	*	typeof	检测操作数类型
	*	void	返回undefined值
	* \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	*	\* / %	乘、除、余
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	*	\+	\-	加、减
	*	\+ 字符串连接 *如果一个操作数是字符串或转化为字符串的的对象，做字符串连接操作，如果两个操作数都不是字符串进行算术加法*
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	*	\<\<	左移
	*	\>\>	有符号右移
	*	\>\>\>	无符号右移
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	*	\< \<= \> \>= 比较数字顺序
	*	\< \<= \> \>= 比较字母表中顺序
	*	instanceof	测试对象类
	*	in	测试对象属性
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	*	== 判断相等
	*	== 判断不等
	*	=== 判断恒等
	*	!== 判断非恒等
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	* &	按位与 
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	* ^	按位异或 
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	* \|	按位或
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	* &&	逻辑与 
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	*	\|\|	逻辑或
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	* ?:	条件运算符
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	*	=	变量或对象赋值
	*	\*= /= %= \+= \-= &= ^= \|= \<\<= \>\>= \>\>\>=	运算且赋值
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	*	,	忽略第一个操作数，返回第二个
8.	eval()
	*	入参不是字符串，返回这个参数；否则把字符串当成js代码编译

#### 语句
1.	表达式语句
2.	复合语句{} 空语句;
3.	声明语句
	*	var
	*	function
4.	条件语句
	*	if / else if / else
	* switch .. case ... defualt
5.	循环
	*	while
	*	do while
	*	for
	* for in
6.	跳转
	*	标签 labelname: statement
	* break	[labelname]
	* continue [labelname]
	* return
	* throw catch finally
7.	其他
	*	with 临时扩展作用于链
	*	debugger 产生一个断点
	*	"use strict"

#### 对象

除了字符串、数字、true、false、null和undefined意外的值都是对象

|对象属性|
|-------------------------|
| 可写 || 能否设置该属性值 |
| 可枚举 || 能否通过for/in循环返回该属性 |
| 可配置 || 能否删除和修改该属性 |
{: rules="groups"}

|对象特性|
|-------------------------|
| 原型prototype || 本对象的属性继承自它的原型对象 |
| 类class || 标识对象类型的字符串 |
| 扩展标记 || 能否向该对象添加新属性 |
{: rules="groups"}

|对象类别和属性|
|-------------------------|
| 内置对象 || 数组、函数、日期、正则表达式 |
| 宿主对象 || 由js解释器嵌入环境(如Web浏览器)定义。表示网页结构的HTMLElement对象均是宿主对象 |
| 自定义对象 || 运行中js代码创建 |
|---------------------|
| 内置对象 || 直接在对象中定义的属性 |
| 宿主对象 || 在对象的原型对象中定义的属性 |
{: rules="groups"}

1.	创建对象
	*	对象直接量
	*	new
	* Object.create()
2.	属性设置和查询
3.	删除属性
4.	检测属性
	*	in hasOwnproperty() propertyIsEnumerable()
	*	!== undefined
5.	枚举属性
6.	getter setter
7.	属性特性 value, writable, enumerable, configurable
	*	Object.getOwnPropertyDescription(obj, name)
	*	Object.defineProperty(obj, name, {...})
	*	Object.defineProperties
8.	对象属性
	* 原型
		*	Object.getPrototypeOf()
		* isPrototypeOf() 类似instanceof运算符
	* class类属性
	* 可扩展
		*	Object.esExtensible()
		*	Object.preventExtensible()
			* Object.seal() Object.isSealed
			*	Object.freeze()	Object.isFrozen
9.	序列化
	*	JSON.stringify JSON.parse

#### 数组
1.	方法
	*	join 将数字中所有元素转换为字符串并拼接，默认用逗号做分隔符
	*	reverse 反序，不创建新数组
	*	sort	排序，默认以元素的字母排序，否则传入一个比较函数
	*	concat 创建并返回一个新数组， 拼接参数到数组中，如果参数是数组则拼接数组元素
	*	slice	返回数组的一个片段，参数指定就爱是和结束位置，不会修改调用数组
	*	splice 插入或删除元素，起始位置，删除元素个数，新元素
	*	push和pop
	* unshift和shift
	* toString和toLocaleString
	*	\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
	* forEach，遍历数组，为每个元素指定调用函数，传递三个参数：元素、索引、数组自身
	* map 返回新数组
	* filter	返回数组子集
	* every some 对数组的逻辑判定
	* reduce reduceRight
	* indexOf lastIndexOf
2.	类型
	*	Array.isArray 判断是否数组

#### 函数

#### 类和模块

#### 正则表达式

#### 子集和扩展

#### 服务器端js

### 客户端js

#### web浏览器
1.	客户端js
	*	window - Window对象，web浏览器的一个窗口或窗体
		*	location
		*	alert()
		*	setTimeOut()
	*	document
		*	getElementId()
	*	事件
2.	HTML嵌入js
	*	\<*script*\>\<*script*\>标签对之间
	*	\<*script*\>的src属性指定的外部文件中
	*	html事件处理程序中 onclick，onmouseover
	*	URL中，使用特殊的"javascript:"
3.	js执行	

#### 	Window对象
1.	计时器
	* setTimeOut() setInterval() 在指定时间后单次或重复调用的函数
	* clearTimeOut()	clearInterval() 取消函数执行
2.	location - Location对象，该窗口中显示的文档的URL
	* window.location === document.location // 始终为true
		* document的URL属性，定位到文档中片段标识符，Location会更新，但是URL不更新
	* 解析URL
		*	href - 字符串，包含URL的完整文本，toString()方法会返回href的值
		*	protocol host hostname port pathname search表示URL的各部分
		*	hash - 返回URL中的片段标识符
		* search - 返回问号之后的URL，通常是某种类型的查询字符串
	*	载入新文档
		*	assign()	窗口载入并显示指定URL的文档
		* replace()	载入新文档前从浏览历史中删除当前文档
		*	reload()	重新载入当前文档 
		* 直接给location赋值，location的URL分解属性亦可写，如search
3.	history
	*	back()
	*	forward()
	*	go() 整形参数，在历史列表中向前（正）或向后（负）跳过任意多页
4.	浏览器navigator和屏幕screen
	*	navigator
		*	appName	Web浏览器全称
		*	appVersion
		*	userAgent	USER-AGENT HTTP头部中发送的字符串，通常包含appVersion中所有信息
		*	platform	操作系统
		*	onLine	是否连接到网络
		*	geolocation	用户地理位置
		*	javaEnable	非标准方法，是否能运行java小程序
		*	cookieEnable	是否可以永久保存cookie
	* screen 提供窗口大小和可用颜色数量的信息
		*	width height 像素为单位的窗口大小
		*	availWidth availHeight 实际可用显示大小
		*	colorDepth	指定显示的BPP，典型值有16，24，32
5.	对话框 - 调试
	*	alert() prompt() confirm()
	*	showModalDialog()
6.	onerror
7.	文档元素id

#### 脚本化文档
Document 文档对象模型DOM - Document Object Model

1.	DOM - 操作HTML文档的基础API
	*	HTML文档的树形表示
2.	选取文档元素
	*	通过id选取，id必须唯一 getElementById()
	*	通过name选取，name不必唯一 getElementsByName()
	*	通过标签名	getElementsByTagName()
	*	通过CSS类选取	getElementsByClassName()
	*	通过CSS选取器	querySelectorAll(), querySelector
3.	文档结构和遍历
	*	Node对象
		*	parentNode
		*	childNodes firstChild lastChild
		*	nextSibling	previousSlibing
		*	nodeType 9-Document 1-Element 3-Text 8-Comment 11-DocumentFragmet
		*	nodeValue
		*	nodeName
4.	属性
	*	HTMLElement定义了通用HTTP属性，特定HTMLElement子元素为其元素定义特定属性
		* js属性名全小小，如果属性包含不止一个单词，除第一个单词外的首字母都大写
		*	如果HTML属性是js的保留字，则加前缀html，如label的for属性->htmlFor
		*	HTML的class属性 -> className
	*	非标准HTML属性
		*	getAttribute() 和 setAttribute()
		*	hasAttribute() 和 removeAttribute()	
5.	Element内容
	*	innerHTML
	*	textContent
6.	滚动
	*	文档坐标和view坐标
	
####	脚本化CSS	

#### 事件处理

|事件分类|
|-------------------------|
| 依赖于设备的输入事件 || 鼠标和键盘 mousedown mousemov mouseup keydown keypress keyup |
|	||	触摸事件 touchmove gesturechange|
| 独立于设备的输入事件 || click |
| 用户界面事件 || focus change submit |
| 状态变化事件 || load |
| 特定api事件 || 拖放api dragstart dragenter dragover drop |
|	||	html5的\<video\> \<audio\>  定义的 waiting playing seeking volumechange |
| 计时器和错误处理 || |	
{: rules="groups"}

1.	事件类型
	*	传统事件类型
		*	表单事件
		*	Window事件
		*	鼠标事件
	*	DOM事件
	*	HTML5事件
	*	触摸屏
2.	注册事件处理程序
	*	给事件目标对象或文档元素设置属性
	*	addEventListrner()	attachEvent()
	
#### 脚本化HTTP
1.	XMLHttpRequest

#### jQuery
1.	基础
	*	jQuery() ($())
		*	传递css选择器(字符串)给$() 返回当前文档中匹配该选择器的元素集
			*	将一个元素或jQuery对象作为第二个参数传递给$()，返回该特定元素的子元素中匹配选择器的部分
		*	传递一个Element、Document、Window对象给$()，将之封装为jQuery对象
		*	传递HTML文本字符串给$()，文本应至少包含一个带<>的HTML标签
			*	jQuery创建HTML元素并封装成jQuery对象返回
			*	可选的第二参数，可接受Document对象来指定创建元素的相关联文档，也可是Object对象
		*	传递函数给$() 文档加载完毕且DOM可操作时传入函数将被调用
		
		

	
	