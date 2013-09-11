---
layout: post
title: "mongoid_4-查询"
description: "mongoid的查询方法"
category: mongoid
tags: [mongoid]
---

参考自[Querying](http://mongoid.org/en/mongoid/docs/querying.html)

###查询
> 所有的查询称为criteria，链式，封装mongodb的动态查询，只在需要时连接数据库
####可查询DSL
> DSL使用Origin::Queryable
	Band.where(name: "Depeche Mode")
	Band.
	  where(:founded.gte => "1980-1-1").
	  in(name: [ "Tool", "Deftones" ]).
	  union.
	  in(name: [ "Melvins" ])
####增强的查询方法
> - Criteria#count
>
>	获取数目, 将会一直连接数据库
	Band.count
	Band.where(name: "Photek").count
> - Criteria#distinct
>
>	获取单个field的所有不同值
	Band.distinct(:name)
	Band.where(:fans.gt => 100000).
	  distinct(:name)
> - Criteria#each
>
>	遍历每个document
	Band.each do |band|
	  p band.name
	end
> - Criteria#exists?
>
>	确定document是否存在
	Band.exists?
	Band.where(name: "Photek").exists?
> - Criteria#find
>
>	通过_id查找document, 找不到抛出错误
	Band.find("4baa56f1230048567300485c")
	Band.find(
	  "4baa56f1230048567300485c",
	  "4baa56f1230048567300485d"
	)
	Band.where(name: "Photek").find(
	  "4baa56f1230048567300485c"
	)
> - Model.find_by
>
>	根据属性查找, 查找不到则抛出错误或者返回nil, 根据raise_not_found_error配置选项
	Band.find_by(name: "Photek")
	Band.find_by(name: "Tool") do |band|
	  band.impressions += 1
	end
> - Criteria#find_or_create_by
>
>	根据属性查找, 查找不到则创建并返回一个新对象
	Band.find_or_create_by(name: "Photek")
	Band.where(:likes.gt => 10).find_or_create_by(name: "Photek")
> - Criteria#find_or_initialize_by
>
>	根据属性查找, 查找不到则初始化并返回一个新对象
	Band.find_or_initialize_by(name: "Photek")
	Band.where(:likes.gt => 10).find_or_create_by(name: "Photek")
> - Criteria#first
>
>	获取第一个document. 如果没有sort选项则使用升序 \_id排序
	Band.first
	Band.where(:members.with_size => 3).first
> - Criteria#first_or_create
>
>	查找第一个document, 没找到则创建并返回一个新的
	Band.where(name: "Photek").first_or_create
> - Criteria#first_or_create!
>
>	查找第一个document, 没找到则创建并返回一个新的, 使用create!创建
	Band.where(name: "Photek").first_or_create!
> - Criteria#first_or_initialize
>
>	查找第一个document, 没找到则初始化并返回一个
	Band.where(name: "Photek").first_or_initialize
> - Criteria#for_js
>
>	根据提供的javascript表达式查找documents, 把javascript包装到`Moped::BSON::Code` 对象中来防止javascript注入攻击
	Band.for_js("this.name = param", param: "Tool")
> - Criteria#last
>
>	获取最后一个document, 没有排序字段则使用升序_id排序
	Band.last
	Band.where(:members.with_size => 3).last
> - Criteria#length / size
>
>	获取存储的document大小, 查询一次后会缓存结构, 后续查询不会再连接数据库
	Band.length
	Band.where(name: "Photek").length
> - Criteria#pluck
>
>	获取所有非空值
	Band.all.pluck(:name)
####预加载
> mongoid在遍历相关documents，根据relations预先加载documents以防止n+1问题，支持除多态belongs_to以外的所有relaition
> Identity Map选项要enable
	class Band
	  include Mongoid::Document
	  has_many :albums
	end
	class Album
	  include Mongoid::Document
	  belongs_to :band
	end
	Band.includes(:albums).each do |band|
	  p band.albums.first.name    # Does not hit the database again.
	end
###查询和存储
> 多documents的inserts, updates, deletion.
>
> - Criteria#create
>
>	创建并存储新文档
	Band.where(name: "Photek").create
> - Criteria#create!
>
>	创建并存储新文档 使用create!
	Band.where(name: "Photek").create!
> - Criteria#build / new
创建一个新文档, 不存储
	Band.where(name: "Photek").build
> - Criteria#update
>
>	更新第一个匹配文档的属性
	Band.where(name: "Photek").update(label: "Mute")
> - Criteria#update_all
>
>	更新所有匹配文档的属性
	Band.where(name: "Photek").update_all(label: "Mute")
> - Criteria#add_to_set
>
>	对所有匹配文档执行$addToSet
	Band.where(name: "Photek").add_to_set(:label, "Mute")
> - Criteria#bit
>
>	对所有匹配文档执行 $bit
	Band.where(name: "Photek").bit(:likes, { and: 14, or: 4 })
> - Criteria#inc
>
>	对所有匹配文档执行 $inc
	Band.where(name: "Photek").inc(:likes, 123)
> - Criteria#pop
>
>	对所有匹配文档执行pop
	Band.where(name: "Photek").pop(:members, -1)
	Band.where(name: "Photek").pop(:members, 1)
> - Criteria#pull
>
>	对所有匹配文档执行$pull
	Band.where(name: "Tool").pull(:members, "Maynard")
> - Criteria#pull_all
>
>	对所有匹配文档执行 $pullAll
	Band.where(name: "Tool").pull_all(:members, [ "Maynard", "Danny" ])
> - Criteria#push
>
>	对所有匹配文档执行 $push
	Band.where(name: "Tool").push(:members, "Maynard")
> - Criteria#push_all
>
>	对所有匹配文档执行$pushAll
	Band.where(name: "Tool").push_all(:members, [ "Maynard", "Danny" ])
> - Criteria#rename
>
>	对所有匹配文档执行 $rename
	Band.where(name: "Tool").rename(:name, :title)
> - Criteria#set
>
>	对所有匹配文档执行 $set
	Band.where(name: "Tool").set(:likes, 10000)
> - Criteria#unset
>
>	对所有匹配文档执行 $unset
	Band.where(name: "Tool").unset(:likes)
> - Criteria#delete
>
>	删除所有匹配文档, 忽略或跳过限制参数
	Band.where(label: "Mute").delete
> - Criteria#destroy
>
>	删除所有匹配文档, 执行回调,
	Band.where(label: "Mute").destroy
###作用域
> 为业务域复用查询语句提供遍历方法
####命名范围
	class Band
	  include Mongoid::Document
	  field :country, type: String
	  field :genres, type: Array
	  scope :english, where(country: "England")
	  scope :rock, where(:genres.in => [ "rock" ])
	end
	Band.english.rock # Get the English rock bands.
> 命名范围支持procs和块来接受参数或方法
	class Band
	  include Mongoid::Document
	  field :name, type: String
	  field :country, type: String
	  field :active, type: Boolean, default: true
	  scope :named, ->(name){ where(name: name) }
	  scope :active, where(active: true) do
	    def deutsch
	      tap |scope| do
	        scope.selector.store("origin" => "Deutschland")
	      end
	    end
	  end
	end
	Band.named("Depeche Mode") # Find Depeche Mode.
	Band.active.deutsch # Find active German bands.
####默认范围
	class Band
	  include Mongoid::Document
	  field :name, type: String
	  field :active, type: Boolean, default: true
	  default_scope where(active: true)
	end
	Band.each do |band|
	  # All bands here are active.
	end
> unscoped不使用默认范围,
	Band.unscoped.where(name: "Depeche Mode")
	Band.unscoped do
	  Band.where(name: "Depeche Mode")
	end
	scoped 重新使用默认范围
> Band.unscoped.where(name: "Depeche Mode").scoped
>
> reload 使用默认范围的model是has_many, has_and_belongs_to_many, or embeds_many 关系的部分, 必须重新加载关系来使范围生效
	class Label
	  include Mongoid::Document
	  embeds_many :bands
	end
	class Band
	  include Mongoid::Document
	  field :active, default: true
	  embedded_in :label
	  default_scoped where(active: true)
	end
	label.bands.push(band)
	label.bands #=> [ band ]
	band.update_attribute(:active, false)
	label.bands #=> [ band ] Must reload.
	label.reload.bands #=> []
> 在和 `Mongoid::Paranoia` 关联时 不能使用默认范围,  否则默认范围会覆盖paranoia范围
####类方法
> model上返回criteria对象的类方法的行为类似scope
	class Band
	  include Mongoid::Document
	  field :name, type: String
	  field :active, type: Boolean, default: true
	  def self.active
	    where(active: true)
	  end
	end
	Band.active
###查找和修改
> MongoDB的$findAndModify 是个原子命令.Mongoid可以直接在model中执行此操作或者传递给一个criteria, 立即执行
	Queue.
	  where(pending: true).
	  asc(:created_at).
	  find_and_modify({ "$set" => { pending: false }}, new: true)
> > 第一个参数是个原子操作, 第二个参数可选，是个hash，支持的选项
>	new: (true|false) 返回更新的document
>	remove: (true|false) 删除更新的document
###Map/Reduce
	map = %Q{
	  function() {
	    emit(this.name, { likes: this.likes });
	  }
	}
	reduce = %Q{
	  function(key, values) {
	    var result = { likes: 0 };
	    values.forEach(function(value) {
	      result.likes += value.likes;
	    });
	    return result;
	  }
	}
	Band.where(:likes.gt => 100).map_reduce(map, reduce).out(inline: true)
> 延迟执行. 直到遍历结果才连接数据库，或者调用一个要强制链接数据库的wrapper
	Band.map_reduce(map, reduce).out(replace: "mr-results").each do |document|
	  p document # { "_id" => "Tool", "value" => { "likes" => 200 }}
	end
> MapReduce的out输出是**必须**的. 没有提供则会抛出错误
>
> \#out的可选的选项包括：
	inline: 1: 不在collection中存储结果.
	replace: "name": 用name指定的collection存储结果，如果有相同document则覆盖
	merge: "name": 用name指定的collection存储结果，如果有相同document则合并
	reduce: "name": 用name指定的collection存储结果，如果有相同document则reduce
> map/reduce API
>
> - MapReduce#out
>
>	指定输出的位置
	Band.map_reduce(m, r).out(inline: 1)
> - MapReduce#counts
>
>	获取map reduce计数 (input, emit, reduce, count).
	Band.map_reduce(m, r).out(inline: 1).counts
> - MapReduce#emitted
>
>	获取 emits 数目.
	Band.map_reduce(m, r).out(inline: 1).emitted
> - MapReduce#finalize
>
>	提供一个在任务结束时执行的函数
	func = %Q{
	  function(key, value) {
	    value.emittedtra = true;
	    return value;
	  }
	}
	Band.map_reduce(m, r).out(inline: 1).finalize(func)
> - MapReduce#input
>
>	获取input数目
	Band.map_reduce(m, r).out(inline: 1).input
> - MapReduce#js_mode
>
>	在js模式下执行map reduce
	Band.map_reduce(m, r).out(inline: 1).js_mode
> - MapReduce#output
>
>	获取output数目
	Band.map_reduce(m, r).out(inline: 1).output
> - MapReduce#reduced
>
>	获取reduces数目
	Band.map_reduce(m, r).out(inline: 1).reduced
> - MapReduce#scope
>
>	设置js全局参数
	Band.map_reduce(m, r).out(inline: 1).scope(field: 10)
> - MapReduce#time
>
>	返回执行次数
	Band.map_reduce(m, r).out(inline: 1).time
###聚合
> Mongoid提供一些方法实现简单聚合，可以在类或者criteria中调用
>
> \#min, \#max \#sum如果提供块，将以类似ruby enumerable的方式处理
>
> \# Use map/reduce, return a float with the max value
	Band.where(:likes.gt => 100).max(:likes)
> \# Use enumerable, returning the document with the max value.
	Band.where(:likes.gt => 100).max do |a,b|
	  a.likes <=> b.likes
	end
> - Criteria#avg
>
>	获取指定field的平均
	Band.avg(:likes)
	Band.where(:likes.gt => 100).avg(:likes)
> - Criteria#max
>
>	获取指定field的最大
	Band.max(:likes)
	Band.where(:likes.gt => 100).max(:likes)
> - Criteria#min
>
>	获取指定field的最小
	Band.min(:likes)
	Band.where(:likes.gt => 100).min(:likes)
> - Criteria#sum
>
>	获取指定field的和
	Band.sum(:likes)
	Band.where(:likes.gt => 100).sum(:likes)
###Geo Near
> Mongoid为MongoDB的$geoNear 命令提供一个DSL.
>
>geo_near \[x, y\]坐标
	Bar.where(:likes.gt => 100).geo_near([ 50, 12 ]).spherical
	Bar.where(:likes.gt => 100).geo_near([ 50, 12 ]).each do |document|
	  p document.geo_near_distance
	end
> - GeoNear#average_distance
>
>	所有documents到指定位置的距离平均值
	Bar.geo_near([ 50, 13 ]).average_distance
	session.command({
	  geoNear: "bars",
	  near: [ 50, 13 ]
	})["stats"]["averageDistance"]
> - GeoNear#distance_multiplier
>
>	所有documents到指定位置的距离乘数
	Bar.geo_near([ 50, 13 ]).distance_multiplier(5012)
	session.command({
	  geoNear: "bars",
	  near: [ 50, 13 ],
	  distanceMultiplier: 5012
	})
> - GeoNear#max_distance
>
>	所有documents到指定位置的最大距离
	Bar.geo_near([ 50, 13 ]).max_distance(100)
	session.command({
	  geoNear: "bars",
	  near: [ 50, 13 ],
	  maxDistance: 100
	})
> - GeoNear#spherical
>
>	计算spheres.
	Bar.geo_near([ 50, 13 ]).spherical
	session.command({
	  geoNear: "bars",
	  near: [ 50, 13 ],
	  spherical: true
	})
> - GeoNear#unique
>
>	唯一document是否返回
	Bar.geo_near([ 50, 13 ]).unique(false)
	session.command({
	  geoNear: "bars",
	  near: [ 50, 13 ],
	  unique: false
	})