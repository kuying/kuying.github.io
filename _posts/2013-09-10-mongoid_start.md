---
layout: post
title: "mongoid_1-概述"
description: "mongoid概述"
category: mongoid
tags: [mongoid]
---


参考自[Getting Start](http://mongoid.org/en/mongoid/docs/installation.html)

###安装
####不使用ActiveRecord
	rails new app_name --skip-active-record'
####或者**手工设置**
	删除	database.yml
> 修改 *app/config/application.rb*
	删除	"rails/all"
	添加	require "action_controller/railtie"
		require "action_mailer/railtie"
		require "active_resource/railtie"
		require "rails/test_unit/railtie"
> 修改*ActiveRecord*配置文件 *app/config/environments/development.rb*
	# config.active_record.mass_assignment_sanitizer = :strict
	# config.active_record.auto_explain_threshold_in_seconds = 0.5
> 修改 *app/config/application.rb*
	# config.active_record.whitelist_attributes = true
####gem
> 添加*mongoid*到*Gemfile*
	gem "mongoid"
> 安装*mongoid*
	$ gem install mongoid
###配置
> Rails 应用中可以自动生成
	$ rails g mongoid:config
> 或修改i配置文件 *app/config/mongoid.yml*
	development:
	  sessions:
	    default:
	      database: mongoid
	      hosts:
	        - localhost:27017
> Mongoid 读取配置，获取环境的顺序: 
>
> > *Rails.env*
> >
> > *RACK_ENV*
> >
> > *MONGOID_ENV*
>使用自定义配置
	Mongoid.load!("path/to/your/mongoid.yml", :production)
>mongoid配置项，参考mongoid.yml 可选配置项
- allow_dynamic_fields(true)

	动态添加未定义的属性，设为false时设置未定义属性值抛出错误.
- identity_map_enabled(false)

	mongoid使用id map存储数据, 查询相同文档的相同单元不需要多次查询数据库
- include_root_in_json(false)

	调用to_json方法时, 包含root document和相关的名字
- include_type_for_serialization(false)

	转换为json或xml格式时包含_type字段.
- preload_models(false)

	预读模块
- protect_sensitive_fields(true)

	去除mongoid对\_id \_type属性的保护
- raise_not_found_error(true)

	查找不到Document时, 抛出 Mongoid::Errors::DocumentNotFound 错误还是返回nil
- skip_version_check(false)

	适用MongoHQ或MongoMachine
- scope_overwrite_exception(false)

	定义和已有方法同名scope,抛出错误
- use_activesupport_time_zone(true)

	在rails应用中将时间转换为本地时间
- use_utc(false)

	使用utc时间

###日志
> 修改log级别,默认日志是关闭的
	module MyApplication
	  class Application < Rails::Application
	    Mongoid.logger.level = Logger::DEBUG
	    Moped.logger.level = Logger::DEBUG
	  end
	end
> 修改日志实例
	Mongoid.logger = Logger.new($stdout)
	Moped.logger = Logger.new($stdout)
###备份设置
> 在*mongoid.yml*的host段下设置备份服务器，默认一致性参数是 :eventual, 修改为 :strong, 则所有信息会发送给主node
	sessions:
	  default:
	  hosts:
	    - repl0.myapp.com:27017
	    - repl1.myapp.com:27017
	    - repl3.myapp.com:27017
	  database: mongoid
	  options:
	  consistency: :strong
###分布式部署
> 使用分布的MongoDB环境，设置shard key
	class Person
	  include Mongoid::Document
	  field :first_name, type: String
	  field :last_name, type: String
	  shard_key :first_name, :last_name
	end
> 在*mongoid.yml*，需要确保hosts指向主机.
	sessions:
	  default:
	    hosts:
	      - mongos.myapp.com:27017
	    database: mongoid
	    options:
	    consistency: :eventual