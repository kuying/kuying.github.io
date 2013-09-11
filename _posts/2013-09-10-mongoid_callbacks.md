---
layout: post
title: "mongoid_8-回调"
description: "mongoid提供的回调方法"
category: mongoid
tags: [mongoid]
---

参考自[Callbacks](http://mongoid.org/en/mongoid/docs/callbacks.html)

###document回调

- after_initialize
- after_build
- before_validation
- after_validation
- before_create
- around_create
- after_create
- after_find
- before_update
- around_update
- after_update
- before_upsert
- around_upsert
- after_upsert
- before_save
- around_save
- after_save
- before_destroy
- around_destroy
- after_destroy

> 

> 建议在回调只应用于横切关注点，如更新后台任务

	class Article
	  include Mongoid::Document
	  field :name, type: String
	  field :body, type: String
	  field :slug, type: String
	  before_create :send_message
	  after_save do |document|
	    # Handle callback here.
	  end
	  protected
	  def send_message
	    # Message sending code here.
	  end
	end

> 另外的语法

	class Article
	  include Mongoid::Document
	  field :name, type: String
	  set_callback(:create, :before) do |document|
	    # Message sending code here.
	  end
	end

###关系回调

- after_add
- after_remove
- before_add
- before_remove
>

> 支持关系: embeds_many, has_many, and has_and_belongs_to_many.

###观察者

> 观察者类响应回调的生命周期, 类似于原始类外的触发

	class ArticleObserver < Mongoid::Observer
	  def after_save(article)
	    Notifications.article("admin@do.com", "New article", article).deliver
	  end
	end

> 观察者实例化后才能被调用
> 
> Rails下Mongoid 

会自动实例化在config/application.rb中注册的观察者

	config.mongoid.observers = :article_observer, :audit_observer

> 观察者会自动映射到同名类
>
> 使用Observer.observe 类方法 设置具体类(Product) 或类的符号 (:product) 来更新映射. 支持多个映射关系

	class AuditObserver < Mongoid::Observer
	  observe :account, :balance
	  def after_update(record)
	    AuditTrail.new(record, "UPDATED")
	  end
	end