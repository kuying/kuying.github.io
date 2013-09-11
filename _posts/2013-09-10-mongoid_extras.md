---
layout: post
title: "mongoid_12-扩展"
description: "mongoid的扩展"
category: mongoid
tags: [mongoid]
---

参考自[Extras](http://mongoid.org/en/mongoid/docs/extras.html)

###缓存
> 如果把所有匹配的document都查询到内存中缓存，在后续遍历中不在查询数据库
	Person.where(first_name: "Franziska").cache
###时间戳
> 提供Mongoid::Timestamps提供created_at和updated_at字段
	class Person
	  include Mongoid::Document
	  include Mongoid::Timestamps
	end
> 只提供创建时间或修改时间
	class Person
	  include Mongoid::Document
	  include Mongoid::Timestamps::Created
	end
	class Post
	  include Mongoid::Document
	  include Mongoid::Timestamps::Updated
	end
> 特定方法不使用时间戳
	person.timeless.save
	Person.timeless.create!
> 短名方式
	class Band
	  include Mongoid::Document
	  include Mongoid::Timestamps::Short # For c_at and u_at.
	end
	class Band
	  include Mongoid::Document
	  include Mongoid::Timestamps::Created::Short # For c_at only.
	end
	class Band
	  include Mongoid::Document
	  include Mongoid::Timestamps::Updated::Short # For u_at only.
	end