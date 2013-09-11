---
layout: post
title: "mongoid_2-文档"
description: "mongodb文档类"
category: mongoid
tags: [mongoid]
---

参考自[Documents](http://mongoid.org/en/mongoid/docs/documents.html)

> Documents是Mongoid的核心类, 任何需操作数据库的类必须
	include Mongoid::Document
> MongoDB中使用BSON来表示Document, Documents能在独立的collections中存储, 也可以多层内嵌到其他Documents中
###存储
> Mongoid默认把documents存储在以类名复数形式命名的collection中
>
> Model类不能用"s"结尾, 这会被认为是复数形式, 将会导致bug
>
> Mongoid使用*ActiveSupport::Inflector#classify*来转换文件名和collection名到类名
>
> 在自定义model或类中指定一条规则
	ActiveSupport::Inflector.inflections do |inflect|
	  inflect.singular("status", "status")
	end
> 或指定存储位置
	class Person
	  include Mongoid::Document
	  store_in collection: "citizens", database: "other", session: "secondary"
	end
> > store_in支持lambdas
	class Band
	  include Mongoid::Document
	  store_in database: ->{ Thread.current[:database] }
	end
> document以BSON形式存储到数据库
###field
>类型
- Array
- BigDecimal
- Boolean
- Date
- DateTime
- Float
- Hash
- Integer
- Moped::BSON::ObjectId
- Moped::BSON::Binary
- Range
- Regexp
- String
- Symbol
- Time
- TimeWithZone
>

> 不指定类型的field, 则会作为object. 并且不进行类型转换, 可以带来性能提升
>
> 可以不指定类型的情形
	*不使用web前端,  并且数值已经转换完成*
	*所有field是String*
> 不能作为动态属性, 必须转换的
	BigDecimal
	Date
	DateTime
	Range
####存/取fileld值
> \# 获取
	person.first_name
	person[:first_name]
	person.read_attribute(:first_name)
> \# 设置
	person.first_name = "Jean"
	person[:first_name] = "Jean"
	person.write_attribute(:first_name, "Jean")
> \# 获取filed值, 返回hash
	person.attributes
> \# 设置
	Person.new(first_name: "Jean-Baptiste", middle_name: "Emmanuel")
	person.attributes = { first_name: "Jean-Baptiste", middle_name: "Emmanuel" }
	person.write_attributes( first_name: "Jean-Baptiste",  middle_name: "Emmanuel" )
####Hash类型Field
> 注意使用的key值, 不能有 .
####默认值
> default: 支持静态值和lambas.
>
> 不以lambda或proc定义的默认值不实时计算，在加载时确定
	class Person
	  include Mongoid::Document
	  field :blood_alcohol_level, type: Float, default: 0.40
	  field :last_drink, type: Time, default: ->{ 10.minutes.ago }
	end
> > lambda或proc中的self指向document实例
	field :wasted_at, type: Time, default: ->{ new_record? ? 2.hours.ago : Time.now }
> > 以proc块作为默认值, Mongoid会在其他属性设置完成后再处理, 如果要提前需设置 pre_processed: true.
####别名 as:  
> 存储时短名，使用时长名
	class Band
	  include Mongoid::Document
	  field :n, as: :name, type: String
	end
####自定义字段序列化
> 3个方法
>
> > 实例方法mongoize 获取一个实例并转换为数据库存储形式
> >
> > 类方法mongoize 从应用程序中获取对象, 并转换为数据库存储格式，应用于setter方法中未传入对象
> >
> > 类方法demongoize从数据库中获取一个实例, 并初始化成自定义类型对象
> >
> > 类方法evolve获取对象并决定在查询中如何转化
>
####保留字
> Mongoid.destructive_fields
####自定义id
> > 默认使用Moped::BSON::ObjectId，可直接覆盖\_id field
	class Band
	  include Mongoid::Document
	  field :name, type: String
	  field :_id, type: String, default: ->{ name }
	end
####动态field
> 默认支持动态field - 允许读写存储没有定义的属性.
>
> 如果属性在document中存在, Mongoid将提供标准的getter setter方法
>
> 如果属性在document中不存在, Mongoid将不getter setter方法, 并执行*method_missing*行为. 此时要使用其他的访问方法: (\[\] and \[\]=) or (read_attribute and write_attribute).
>
###本地化field
	class Product
	  include Mongoid::Document
	  field :description, localize: true
	end
> 设置localize后，Mongoid将使用hash对保存locale/value, 但是正常访问则类似String
> 通过对应的_translations方法读取所有值对
####回退Fallbacks
> 使用fallbacks, 某个translation不可用时,Mongoid将自动调用fallback
> Rails 在环境中
	fallbacks config.i18n.fallbacks = true
> 非rails环境, 必须在I18n gem中引用 fallbacks模块
	require "i18n/backend/fallbacks"
	I18n::Backend::Simple.send(:include, I18n::Backend::Fallbacks)
> 定义fallback后, 如果一个translation不存在,  Mongoid将按照定义顺序使用locale代替
	::I18n.fallbacks[:de] = [ :de, :en, :es ]
####查询
> mongoid自动使用当前locale执行查询
####索引
> 为每个需要查找的locale单独定义索引
####验证
> presence validator 所有locale有值
###脏标记
> Mongoid支持跟踪fields修改, 一个被定义的field被修改后将会标记并提供方法查询之
####观察修改
> 改变从document初始化（新建或从数据库中获取）时开始记录，保存操作后清除
> 方法 
	changed?
	changed?
	changed
	changes
	xxx_changed?
	xxx_changed
	xxx_changes
####重置修改
> reset方法, reset_xxx!
> 存储 Mongoid自动更新和查看 changes, 

如果没有changes发生则调用Model#save也不操作数据库
####观察前面的改变
> document存储后通过 Model#previous_changes 观察save前发生的改变
###安全
> 禁止在块操作中修改属性 attr_protected attr_accessible
	attr_protected 定义保护名单, 所有其他fields将不能set
	attr_accessible 定义白名单, 名单外不能访问
> 提供role权限, as
>
> 在new或create方法中传入as option
	class User
	  include Mongoid::Document
	  field :first_name, type: String
	  field :password, type: String
	  attr_accessible :first_name, as: [ :default, :admin ]
	end
	
	Person.new({ first_name: "Corbin" }, as: :admin)
	Person.create({ first_name: "Corbin" }, as: :default)
	Person.create!({ first_name: "Corbin" }, as: :admin)
> 如果需要设置被保护字段，在独立调用中覆写，传递一个block给document手工设置
	Person.new(first_name: "Corbin") do |person|
	  person.password = "password"
	end
###只读属性
> attr_readonly, update或remove时触发ReadonlyAttribute error 
###继承
> 在根和子document中都支持继承, fields relations validations 和scopes都被复制到子类中
>
> 子类存储在父类的collection中, 使用_type区分
>
> 关联
>
> has one或has many的关联，通过 set 或者 build方法 create 方法关联到任意子类