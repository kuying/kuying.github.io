---
layout: post
title: "mongoid_6-嵌套属性"
description: "mongoid的属性嵌套"
category: mongoid
tags: [mongoid]
---

参考自[Nested Attributes](http://mongoid.org/en/mongoid/docs/nested_attributes.html)

> 支持单一操作中更新Document和Document的关系
###公共行为
>可用于任何关系，添加accepts_nested_attributes_for属性后, 相关relation会自动保存
	class Band
	  include Mongoid::Document
	  embeds_many :albums
	  belongs_to :producer
	  accepts_nested_attributes_for :albums, :producer
	end
> 实现嵌套属性后, 将提供 relation_attribute=方法, 来更新子文档属性, 并可以作为属性使用, 支持 update_attributes, update_attributes! attributes=等
	band = Band.first
	band.producer_attributes = { name: "Flood" }
	band.attributes = { producer_attributes: { name: "Flood" }}
###操作
####1-1 Operations
> 包括*embeds_one*, *embedded_in*, *has_one*, and *belongs_to*.
#####设置
> 语法
	album.producer_attributes = { name: "Flood" }
	album.attributes =
	  { producer_attributes: { name: "Flood" }}
> 定义
	class Album
	  include Mongoid::Document
	  belongs_to :producer
	  accepts_nested_attributes_for :producer
	end
#####更新, 提供id或_id
> 语法
	album.producer_attributes =
	  { id: ..., name: "Flood" }
	album.attributes = {
	  producer_attributes: { id: ..., name: "Flood" }
	}
> 定义
	class Album
	  include Mongoid::Document
	  belongs_to :producer
	  accepts_nested_attributes_for :producer
	end
#####不匹配一定的criteria则拒绝
> 语法
	album.producer_attributes = { name: "Flood" }
	album.attributes = {
	  producer_attributes: { name: "Flood" }
	}
> 定义
	class Album
	  include Mongoid::Document
	  belongs_to :producer
	  accepts_nested_attributes_for :producer,
	    reject_if: ->(attrs){ attrs[:name] == "Flood" }
	end
#####如果所有field为空则拒绝
> 语法
	album.producer_attributes =
	  { name: "", label: "" }
	album.attributes = {
	  producer_attributes = { name: "", label: "" }
	}
> 定义
	class Album
	  include Mongoid::Document
	  belongs_to :producer
	  accepts_nested_attributes_for :producer,
	    reject_if: :all_blank
	end
#####删除1-1关系, 所有属性必须匹配并 \_destory必须为真
> 语法
	album.producer_attributes =
	  { name: "Flood", _destroy: "1" }
	album.attributes = {
	  producer_attributes:
	    { name: "Flood", _destroy: "1" }
	}
> 定义
	class Album
	  include Mongoid::Document
	  belongs_to :producer
	  accepts_nested_attributes_for :producer,
	    allow_destroy: true
	end
#####更新, 不创建
> 语法
	album.producer_attributes =
	  { id: ..., name: "Flood" }
	album.attributes = {
	  producer_attributes:
	    { id: ..., name: "Flood" }
	}
> 定义
	class Album
	  include Mongoid::Document
	  belongs_to :producer
	  accepts_nested_attributes_for :producer,
	    update_only: true
	end
####1-n/n-n 操作
> 包括embeds_many, has_many and has_and_belongs_to_many
#####创建新document
> 语法
	band.albums_attributes = {
	  "0" => { name: "Violator" }
	}
	band.albums_attributes = [
	  { name: "Violator" }
	]
	band.attributes = {
	  albums_attributes: {
	    "0" => { name: "Violator" }
	  }
	}
> 定义
	class Band
	  include Mongoid::Document
	  embeds_many :albums
	  accepts_nested_attributes_for :albums
	end
#####限制新建子对象数目, 超出则抛出错误
> 语法
	band.albums_attributes = {
	  "0" => { name: "Violator" },
	  "1" => { name: "101" },
	  "2" => { name: "Music for the Masses" }
	}
	band.attributes = {
	  albums_attributes: {
	    "0" => { name: "Violator" },
	    "1" => { name: "101" },
	    "2" => { name: "Music for the Masses" }
	  }
	}
> 定义
	class Band
	  include Mongoid::Document
	  embeds_many :albums
	  accepts_nested_attributes_for :albums, limit: 2
	end
#####更新, 需提供每个document的id或\_id
> 语法
	band.albums_attributes = {
	  "0" => { id: ..., name: "Violator (Edit)" },
	  "1" => { id: ..., name: "101 (Edit)" }
	}
	band.albums_attributes = [
	  { id: ..., name: "Violator (Edit)" },
	  { id: ..., name: "101 (Edit)" }
	]
	band.attributes = {
	  albums_attributes: {
	    "0" => { id: ..., name: "Violator (Edit)" },
	    "1" => { id: ..., name: "101 (Edit)" }
	  }
	}
> 定义
	class Band
	  include Mongoid::Document
	  embeds_many :albums
	  accepts_nested_attributes_for :albums
	end
#####删除1个文档, 需要给定id或\_id, \_destory要为真
> 语法
	band.albums_attributes = {
	  "0" => { id: ..., _destroy: "1" },
	}
	band.albums_attributes = [
	  { id: ..., _destroy: "1" },
	]
	band.attributes = {
	  albums_attributes: {
	    "0" => { id: ..., _destroy: "1" },
	  }
	}
> 定义
	class Band
	  include Mongoid::Document
	  embeds_many :albums
	  accepts_nested_attributes_for :albums,
	    allow_destroy: true
	end
#####不匹配指定criteria则拒绝
> 语法
	band.albums_attributes = {
	  "0" => { name: "Violator" }
	}
	band.albums_attributes = [
	  { name: "Violator" }
	]
	band.attributes = {
	  albums_attributes: {
	    "0" => { name: "Violator" }
	  }
	}
> 定义
	class Band
	  include Mongoid::Document
	  embeds_many :albums
	  accepts_nested_attributes_for :albums,
	    reject_if: ->(attrs){ attrs[:name] == "Violator" }
	end
#####全空白则拒绝
> 语法
	band.albums_attributes = {
	  "0" => { name: "" }
	}
	band.albums_attributes = [
	  { name: "" }
	]
	band.attributes = {
	  albums_attributes: {
	    "0" => { name: "" }
	  }
	}
> 定义
	class Album
	  include Mongoid::Document
	  embeds_many :albums
	  accepts_nested_attributes_for :albums,
	    reject_if: :all_blank
	end