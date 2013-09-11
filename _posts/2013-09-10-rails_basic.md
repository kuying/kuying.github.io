---
layout: post
title: "rails_1-基础"
description: "ruby、Restful、rails基础"
category: rails
tags: [rails]
---


###rails简介

***

MVC的网站框架

- ORM面向对象的方法操作数据库
- URL路由

Rails哲学

- DRY Don't Repeat Yourself 不要重复自己
- Convection Over Configuration 约定大于配置
- REST

开发环境
> 如果出现UTF-8中文, 建议在文件头加上
	#encoding: utf-8

###ruby简介

***

- dynamic type 动态类型
- everything is object 一切皆对象
- code block 块编程
- DSL

####rubyGems
Ruby的库的管理系统, [ruby toolbox](https://www.ruby-toolbox.com/)推荐常用库

> 常用命令
	gem -v 查询RubyGems版本
	gem update --system 升级RubyGems版本
	gem install gem_name 安装gem, --no-ri --no-rdoc不安装文档
	gem list 列出所有安装套件
	gem update gem_name 更新gem到最新版本
	gem update 更新所有安装的gems
	gem install -v x.x.x gem_name 安装特定版本
	gem uninstall gem_name 卸载
####语法
#####基本类型
	Fixnum 整数
	Float 浮点数
	String 字符串
#####一切皆对象
>没有全局函数,所有的方法都是对象的方法
#####局部变量
>小写字母开头, 单词之间用_分隔
#####类型转换
	to_s
	to_i
	to_f
#####常数
>大写字母开头
#####空值nil
>表示未设定, 未定义的状态
#####注释
>偏好单行注释, 使用#
>
>多行注释比较少见
	=begin
	  comment
	=end
#####队列Array
>使用\[\], 索引从0开始, 队列元素可以是异构的
#####散列Hash
>使用\{\}, 键值对
#####符号Symbols
>唯一且不变的识别名称
> 使用symbol可以获得效率上的提升, 相同名称的symbol不会重复构造
#####控制流
比较

	1 > 2 # 大於
	1 < 2 # 小於
	5 >= 5 # 大於等於
	5 <= 4 # 小於等於
	1 == 1 # 等於
	2 != 1 # 不等於
     	
	( 2 > 1 ) && ( 2 > 3 ) # 和
	( 2 > 1 ) || ( 2 > 3 ) # 或
控制结构
> if
	if ... elsif ... else
	if unless放到行末
>三元运算
	... ? ... : ...
>case
	case value
	  when condition
	    expression
	  else
	    expression
	end
循环结构
>while
	i=0
	while ( i < 10 )
	  i += 1
	  next if i % 2 == 0 #跳过双数, next类似continue
	end
>until
	i = 0
	i += 1 until i > 10
	puts i # 输出11
>loop
	i = 0
	loop do 
	  i += 1
	  break if i > 10 # 中断
	end
>使用next break控制循环过程
>一般很少使用循环结构, 使用迭代器替换之
#####真假
>只有**false**和**nil**是假, 其他都是真
#####正则表达式
>匹配 =~
#####方法定义
>def ... end
>
>ruby返回最后一行计算的值
>
>调用方法时, 可以省略括号
#####?和!的惯例
>使用?结尾表示返回Boolean值
>
>使用!结尾表示有某种副作用
#####类
>类是一种常数, 大写开头,使用new方法生成对象
>
>定义类别
>
>构造方法initialize
>
>实例方法和类方法(self.xxxx)
>
>对象变量@xxx 和类变量 @@xxx 类外无法存取
######类定义内可以执行方法
	class Demo
      puts "foobar"
	end
>主要用于元编程
######方法封装
>默认public, 声明private或protected后,则后面的方法会套用
>
>private和protected在整个继承体系内斗能被调用, 差别在
	private方法不能指定接收者,预设讲self作为private的接收者
	private方法除了被一个类或子类对象呼叫, 也可以让另一个相同类的对象来呼叫
> object.call_method可以改写为 object.\_\_send\_\_(:call\_method)
######继承
>使用<代表类继承
>
>子类调用super表示被覆盖的父类方法
>
>没有括号的 super 表示传递所有参数, 有括号需要自己指定参数
#####迭代器&块
	do ... end
	{}
#####yield
>方法中使用yield方法可执行colde block的参数
	def call_block
	  puts "Start"
	  yield
	  yield
	  puts "End"
	end
	
	call_block { puts "Blocks are cool!" }
	# 输出
	# "Start"
	# "Blocks are cool!"
	# "Blocks are cool!"
	# "End"
>带参数的code block
	def call_block
	  yield(1)
	  yield(2)
	  yield(3)
	end
	
	call_block { |i|
	  puts "#{i}: Blocks are cool!"
	}
	# 输出
	# "1: Blocks are cool!"
	# "2: Blocks are cool!"
	# "3: Blocks are cool!"
#####Proc对象
讲block块转换为一个参数
	def call_block(&block)
	    block.call(1)
	    block.call(2)
	end
	
	call_block { |i| puts "#{i}: Blocks are cool!" }
	
	proc_1 = Proc.new { |i| puts "#{i}: Blocks are cool!" }
	call_block(&proc_1)
	
	proc_2 = lambda { |i| puts "#{i}: Blocks are cool!" }
	call_block(&proc_2)
#####传递变参 \*
	def my_sum(*val)
	  val.inject(0) { |sum, v| sum + v }
	end
	
	puts my_sum(1,2,3,4) # val 包含所有参数的数组  [1,2,3,4]
	# 输出 10
#####参数末尾的Hash可以省略{}
####异常处理
	begin
	  ...
	rescue 
	  ...
	end
>抛出异常
	raise
####Module
>类似Class但是不能new
- 当做Namespace放一下工具方法
- Mixins, 将一个Module插入道一个类中,此类就会拥有Module的方法

####元编程
>*define_method*
####反射机制
>动态知道对象的信息
	Object.methods
	Object.respon_to? :xxx
####||=
	reslt ||= a
	# 如果result是nil或false,将a指派给result

###CRUD	
***
Create Read Update Delete四项基本数据操作

new create index show edit update destroy七大方法
###Restful
***
两大核心精神
>使用Resource来作为识别的资源,使用一个URL网址来代表一个Resource => 路由
>同一个Resource有不同的Representations的格式变化 => respond_to

HTTP方法(HTTP 1.1)
>HEAD、GET、POST、PUT、DELETE、TRACE、OPTIONS、CONNECT、PATCH

####PathHelper的对应关系
	model_path(@model)	GET:show			PUT:update	DELETE:destroy
	models_path		GET:index	POST:create
	edit_model_path(@model)	GET:edit
	new_model_path		GET:new
####respond_to
在一个action中支持不同的资料格式
