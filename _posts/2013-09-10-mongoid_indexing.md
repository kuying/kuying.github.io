---
layout: post
title: "mongoid_10-索引"
description: "mongoid的索引"
category: mongoid
tags: [mongoid]
---

参考自[Indexing](http://mongoid.org/en/mongoid/docs/indexing.html)

> 使用index给文档定义一个有方向的索引, 可以用作hash参数
>
> 提升检索效率
	class Person
	  include Mongoid::Document
	  field :ssn
	  index({ ssn: 1 }, { unique: true, name: "ssn_index" })
	end
> 可以给embedded对象直接索引
	class Person
	  include Mongoid::Document
	  embeds_many :addresses
	  index "addresses.street" => 1
	end
> 可以给多个fieldd联合索引
	class Person
	  include Mongoid::Document
	  field :first_name
	  field :last_name
	  index({ first_name: 1, last_name: 1 }, { unique: true })
	end
> 索引可以是稀疏的
	class Person
	  include Mongoid::Document
	  field :ssn
	  index({ ssn: -1 }, { sparse: true })
	end
> 索引可以在后台运行
	class Person
	  include Mongoid::Document
	  field :ssn
	  index({ ssn: 1 }, { unique: true, background: true })
	end
> 对于定位的索引, 确保对应的filed是数组
	class Person
	  include Mongoid::Document
	  field :location, type: Array
	  index({ location: "2d" }, { min: -200, max: 200 })
	end
> 索引能够指定数据库的范围
	class Person
	  include Mongoid::Document
	  field :ssn
	  index({ ssn: 1 }, { database: "users", unique: true, background: true })
	end
> 使用mongoid定义外键索引
	class Comment
	  include Mongoid::Document
	  belongs_to :post, index: true
	  has_and_belongs_to_many :preferences, index: true
	end
> 使用rake任务在数据库中创建索引
	rake db:mongoid:create_indexes
> 删除
	rake db:mongoid:remove_indexes