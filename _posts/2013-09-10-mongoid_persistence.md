---
layout: post
title: "mongoid_3-存储"
description: "mongoid的持久化"
category: mongoid
tags: [mongoid]
---

参考自[Persistence](http://mongoid.org/en/mongoid/docs/persistence.html)

> Mongoid支持CRUD操作. monogoid只对修改了的field进行原子更新
###标准方法
> - Model.create
>
>	插入一条或多条document到数据库
	Person.create(
	  first_name: "Heinrich",
	  last_name: "Heine"
	  )
	Person.create([
	  { first_name: "Heinrich", last_name: "Heine" },
	  { first_name: "Willy", last_name: "Brandt" }
	  ])
	Person.create(first_name: "Heinrich") do |doc|
	  doc.last_name = "Heine"
	end
> - Model.create!
>
>	插入一条或多条document到数据库, 验证失败时抛出错误
	Person.create!(
	  first_name: "Heinrich",
	  last_name: "Heine"
	)
	Person.create!(first_name: "Heinrich") do |doc|
	  doc.last_name = "Heine"
	end
> - Model#save
>
>	保存修改或插入记录，通过new_record?判断是否新纪录，可通过validate: false跳过校验
	person = Person.new(
	  first_name: "Heinrich",
	  last_name: "Heine"
	)
	person.save
	person.save(validate: false)
	person.first_name = "Christian Johan"
	person.save
> - Model#save!
>
>	保存修改或插入记录. 验证失败抛出错误
	person = Person.new(
	  first_name: "Heinrich",
	  last_name: "Heine"
	)
	person.save!
	person.first_name = "Christian Johan"
	person.save!
> - Model#update_attributes
>
>	更新属性
	person.update_attributes(
	  first_name: "Jean",
	  last_name: "Zorg"
	)
> - Model#update_attributes!
>
>	更新属性, 校验失败抛出错误
	person.update_attributes!(
	  first_name: "Jean",
	  last_name: "Zorg"
	)
> - Model#update_attribute
>
>	更新单个属性, 跳过校验
	person.update_attribute(:first_name, "Jean")
> - Model#upsert
>
>	update/insert, 如果document存在, 则覆盖, 否则插入.
> 
>	仅支持 {before|after|around}_upsert 回调.
	person = Person.new(
	  first_name: "Heinrich",
	  last_name: "Heine"
	)
	person.upsert
> - Model#touch
>
>	更新document的 updated_at 时间戳, 并且级联touch所有belongs_to关系
>
>	此操作跳过校验并且没有回调
	person.touch
	person.touch(:audited_at)
> - Model#delete
>
>	删除document, 没有回调
	person.delete
> - Model#destroy
>
>	调用destroy回调, 删除document.
	person.destroy
> - Model.delete_all
>
>	删除数据库中所有匹配提供属性的document 没有回调
	Person.delete_all
	Person.delete_all(first_name: "Heinrich")
> - Model.destroy_all
>
>	删除数据库中所有匹配提供属性的document 调用destroy回调
	Person.destroy_all
	Person.destroy_all(first_name: "Heinrich")
###原子存储方法
> 不支持回调, 不支持校验

- Model#add_to_set 添加一个值到数组中，不存在才添加

	person.add_to_set(:aliases, "Bond")	
- Model#bit 位操作 针对Integer

	person.bit(:age, { and: 10, or: 12 })
- Model#inc 针对数字field 增加value值

	person.inc(:age, 1)
- Model#pop 删除数组的第一或最后一个元素，参数只能是1或-1

	person.pop(:aliases, 1)
- Model#pull 从数组field中删除一个等于value的

	person.pull(:aliases, "Bond")
- Model#pull_all 删除多个值

	person.pull_all(:aliases, \[ "Bond", "James" \])
- Model#push 添加元素到field数组中，field不存在则新增一个数组类型

	person.push(:aliases, "007")
- Model#push_all 添加多个

	person.push_all(:aliases, \[ "007", "008" \])
- Model#rename 修改field名

	person.rename(:bday, :dob)
- Model#set 相当于sql中的set field = value语法，设置值对

	person.set(:name, "Tyler Durden")
- Model#unset 删除字段

	person.unset(:name)
###自定义
####Model级选项
> store_in，指定存储的collection database session
	class Band
	  include Mongoid::Document
	  store_in collection: "artists", database: "music", session: "secondary"
	end
####运行时选项
>  with，with是一次性操作
> 
> 在store, query, update, or remove操作时使用 #with
	Band.with(database: "music-non-stop").create
	Band.with(collection: "artists").delete_all
	band.with(session: :tertiary).save!
####数据库级别
> 整个运行时有效Mongoid.override_session和 Mongoid.override_database.
>
> 常用在国际化应用中, 数据存储在不同locale 不同session中, 但是数据结构一致.
	class BandsController < ApplicationController
	  before_filter :switch_database
	  after_filter :reset_database
	  private
	  def switch_database
	    I18n.locale = params[:locale] || I18n.default_locale
	    Mongoid.override_database("my_db_name_#{I18n.locale}")
	  end
	  def reset_database
	    Mongoid.override_database(nil)
	  end
	end
> \#with可用来暂时修改安全模式 .
	Band.with(safe: true).create
	Band.with(safe: { w: 3 }).create!
> > 能够传递给safe的参数:
>	true:  安全模式存储，没有其他选项
>	false:  非安全模式存储
>	fsync: true|false: 是否提供 fsync.
>	w: n:  返回前写入的nodes数目
>	wtimeout: n: 写入多个nodes的timeout值
####Session 与 Collection访问
	Band.mongo_session
	band.mongo_session
	Band.collection
	band.collection
	Band.mongo_session.with(safe: false, database: "musik") do |session|
	  session[:artists].find(...)
	end
#####Collections限制
> 默认不限制collections
>
> Moped创建
	session.command(create: "name", capped: true, size: 10000000, max: 1000)
> Mongo控制台创建
	db.createCollection("name", { capped: true, size: 10000000, max: 1000 });