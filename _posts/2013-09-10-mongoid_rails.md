---
layout: post
title: "mongoid_11-Rails支持"
description: "mongoid的rails支持"
category: mongoid
tags: [mongoid]
---

参考自[Rails](http://mongoid.org/en/mongoid/docs/rails.html)

###多参数属性
> 如date、time等类型，默认不支持
	class Person
	  include Mongoid::Document
	  include Mongoid::MultiParameterAttributes
	end
###Railties
####配置
> 设置到application.rb，覆盖config/mongoid.yml
	module MyApplication
	  class Application < Rails::Application
	    config.mongoid.logger = Logger.new($stdout, :warn)
	    config.mongoid.persist_in_safe_mode = true
	  end
	end
####model预读
> 为支持model继承环境下，mongoid需要将所有model预先读取（development环境），不使用继承可关闭之
	config.mongoid.preload_models = false
####异常	
> mongoid通知rails返回指定的http状态码
	Mongoid::Errors::DocumentNotFound : 404
	Mongoid::Errors::Validations : 422
###Rake任务
> mongoid为rails 3提供下列任务
- db:create: do nothing.
- db:create_indexes: 读取所有model的index定义并在数据库中创建之.
- db:remove_indexes: 读取所有model的index定义并在数据库中删除之.
- db:drop: 删除所有collections.
- db:migrate: do nothing.
- db:purge: 删除所有数据，包括index.
- db:schema:load: do nothing.
- db:seed: 从db/seeds.rb seed数据
- db:setup: 创建indexs并seed数据.
- db:test:prepare: do nothing.
	