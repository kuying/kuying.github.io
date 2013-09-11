---
layout: post
title: "mongoid_7-索引映射"
description: "mongoid的索引映射机制"
category: mongoid
tags: [mongoid]
---

参考自[Identity Map](http://mongoid.org/en/mongoid/docs/identity_map.html)

###设置
	修改mongoid.yml. 设置identity_map_enabled: true
	或在代码中设置 Mongoid.identity_map_enabled = true
>当document从数据库中读取后, 会根据类和id自动添加到索引映射中, 随后根据其id过来的请求将不会再连接数据库除非索引映射自动保存回数据库, 查询不到索引映射不保存nil数据
>
>对于find查询将先查找索引映射，查不到则查找数据库
###工作单元
>为防止数据陈旧, 索引映射中的document只能在一个单独的工作单元中存在, 一个工作单元通常是应用程序的一次请求
>
>Rails 不需要做额外工作,  mongoid自动清理索引映射在每次请求后
>
>
> 临时禁止索引映射 强制从数据库中读取document
	# 当前线程下禁止索引映射
	Mongoid.unit_of_work(disable: :current) do
	  Band.find(1)
	end
	# 禁止所有线程下索引映射
	Mongoid.unit_of_work(disable: :all) do
	  Band.find(1)
	end
####Rake任务和后台工作
> 在后台任务和rake时不要使用索引映射, >
>
>除非能确切知道哪些documents将被加载和内存消费,并且将任务和rake放到一个工作单元中
	desc "A long running rake task"
	task "db:migrate_data" => :environment do
	  Mongoid.unit_of_work(disable: :all) do
	    # Do my work here.
	  end
	end
	class BackgroundJob
	  @queue = :background
	  def self.perform(id)
	    Mongoid.unit_of_work(disable: :all) do
	      # Do work here.
	    end
	  end
	end
###测试
> 如果使用索引映射 在每个测试执行前需清理
	RSpec.configure do |config|
	  config.before(:each) { Mongoid::IdentityMap.clear }
	end