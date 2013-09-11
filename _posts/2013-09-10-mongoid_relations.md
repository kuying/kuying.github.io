---
layout: post
title: "mongoid_5-关系"
description: "mongoid数据model之间的关系"
category: mongoid
tags: [mongoid]
---

参考自[Relations](http://mongoid.org/en/mongoid/docs/relations.html)

###关系
> 表示在domain和数据库中model和model之间的关联
	Embedded关系 一个document存储在另外一个document中
	Referenced关系 documents通过存储外键(id)来引用存储在另一个collection中的documents
###公共行为
####属性
> 所有关系包含一个target, 指向代理的document或documents, 一个base持有关系的doument, 一个元数据说明关系的信息.
	class Person
	  include Mongoid::Document
	  embeds_many :addresses
	end
	person.addresses = [ address ]
	person.addresses.target # returns [ address ]
	person.addresses.base # returns person
	person.addresses.metadata # returns the metadata
####扩展
> 所有关系都支持应用来指定具体功能的方法：定义关系时提供一个block
	class Person
	  include Mongoid::Document
	  embeds_many :addresses do
	    def find_by_country(country)
	      where(country: country).first
	    end
	    def chinese
	      @target.select { |address| address.country == "China"}
	    end
	  end
	end
	person.addresses.find_by_country("Mongolia") # returns address
	person.addresses.chinese # returns [ address ]
####自定义关系名
> 可以给关系任意命名，但是要确保Mongoid能从名字推断出类，并推断出另一侧关系名称
	class Lush
	  include Mongoid::Document
	  embeds_one :whiskey, class_name: "Drink", inverse_of: :alcoholic
	end
	class Drink
	  include Mongoid::Document
	  embedded_in :alcoholic, class_name: "Lush", inverse_of: :whiskey
	end
####验证
> 默认时, Mongoid将验证任何关系的子类，通过validates_associated将之加载至内存:
	embeds_many
	embeds_one
	has_many
	has_one
	has_and_belongs_to_many
> 不需要则在定义时将之关闭
	class Person
	  include Mongoid::Document
	  embeds_many :addresses, validate: false
	  has_many :posts, validate: false
	end
####多态
> 如果一个被内嵌的子document 能够belong to多种父document， 可以在父document中增加as选项，在子document中增加polymorphic选项
>
> 这样在子对象中会新建一个field存储父document类型
>
> 除了has_and_belongs_to_many 都支持此属性
	class Band
	  include Mongoid::Document
	  embeds_many :photos, as: :photographic
	  has_one :address, as: :addressable
	end
	class Photo
	  include Mongoid::Document
	  embedded_in :photographic, polymorphic: true
	end
	class Address
	  include Mongoid::Document
	  belongs_to :addressable, polymorphic: true
	end
####层叠回调
> 需要在存储父document操作时，同时执行内嵌的document的回调，需要提供cascade callbacks选项，只支持 embeds_one 和 embeds_many关系.
	class Band
	  include Mongoid::Document
	  embeds_many :albums, cascade_callbacks: true
	  embeds_one :label, cascade_callbacks: true
	end
	band.save # Fires all save callbacks on the band, albums, and label.
####依赖 dependent
> 指示Mongoid 在关系的一侧删除后或尝试删除的处理 仅支持referenced关系
	:delete: 删除子document
	:destroy: 删除子 document.
	:nullify: 保留子 document. 默认
	:restrict: 抛出错误如果子document非空

	class Band
	  include Mongoid::Document
	  has_many :albums, dependent: :delete
	  belongs_to :label, dependent: :nullify
	end
	class Album
	  include Mongoid::Document
	  belongs_to :band
	end
	class Label
	  include Mongoid::Document
	  has_many :bands, dependent: :restrict
	end
	label = Label.first
	label.bands.push(Band.first)
	label.delete # Raises an error since bands is not empty.
	Band.first.delete # Will delete all associated albums.
####自动保存
> Mongoid不同于Active Record, 不会自动保存子关系
>
> 需要自动保存关联关系, (embedded除外, 已经是数据库的一部分) 要添加autosave选项
>
> 执行accept_nested_attributes_for 或验证关系存在性 autosave动作会被自动关联到关系上
	class Band
	  include Mongoid::Document
	  has_many :albums, autosave: true
	end
	band = Band.first
	band.albums.build(name: "101")
	band.save #=> Will save the album as well.
####递归嵌套
> document能够递归嵌套自身recursively_embeds_one 或 recursively_embeds_many, 提供parent_xxx 和 child_xxx 方法访问
>
> 只支持embedded关系
	class Tag
	  include Mongoid::Document
	  recursively_embeds_many
	end
	root = Tag.new(name: "programming")
	child_one = root.child_tags.build
	child_two = root.child_tags.build
	root.child_tags # [ child_one, child_two ]
	child_one.parent_tag # [ root ]
	child_two.parent_tag # [ root ]
	class Node
	  include Mongoid::Document
	  recursively_embeds_one
	end
	root = Node.new
	child = Node.new
	root.child_node = child
	root.child # child
	child.parent_node # root
####存在谓词
> name? 和 has_name? 来检查关系是否有效
	class Band
	  include Mongoid::Document
	  embeds_one :label
	  embeds_many :albums
	end
	band.label?
	band.has_label?
	band.albums?
	band.has_albums?
####自动创建 autobuild
> 1对1的关系 (embeds_one, has_one) 有一个自动创建选项，当访问关系并且为nil时自动创建之
>
> 存在谓词不会触发自动生成, 如果document不存在会返回失败
	class Band
	  include Mongoid::Document
	  embeds_one :label, autobuild: true
	  has_one :producer, autobuild: true
	end
	band = Band.new
	band.label # Returns a new empty label.
	band.producer # Returns a new empty producer.
####Touching
> belongs_to关系, 使用:touch选项, 调用子document的touch方法 将触发父document调用#touch
	class Band
	  include Mongoid::Document
	  belongs_to :label, touch: true
	end
	band = Band.first
	band.touch #=> Calls touch on the parent label.
###元数据
> 包含关系的信息，便于扩展
	# 从类或document中通过name获取元数据
	Model.reflect_on_association(:relation_name)
	model.reflect_on_association(:relation_name)
	# 从当前对象获取元数据
	model.metadata
	# 从指定对象的指定关系中获取元数据
	person.addresses.metadata
####元数据对象
- Metadata#as 返回父类名
- Metadata#as? 返回as选项是否存在
- Metadata#autobuilding? 返回关系是否autobuilding
- Metadata#autosaving? 返回关系是否 autosaving.
- Metadata#cascading_callbacks? 返回返回关系是否有级联回调
- Metadata#class_name 返回代理document的类名
- Metadata#cyclic? 返回关系是否一个循环关系
- Metadata#dependent 返回关系的dependent选项
- Metadata#dependent? 返回关系是否 有dependent
- Metadata#destructive? 返回关系是否有 dependent  delete 或 destroy.
- Metadata#embedded? 返回关系是否embedded在其他document
- Metadata#forced_nil_inverse? 返回关系是否有nil inverse定义
- Metadata#foreign_key 返回是否有外键定义
- Metadata#foreign_key_check 返回外键field 的 dirty check 方法
- Metadata#foreign_key_setter 返回外键field的setter方法
- Metadata#indexed? 返回外键是否自动索引
- Metadata#inverses 返回所有逆向关系名
- Metadata#inverse 返回单个关系的逆向关系名
- Metadata#inverse_class_name 返回关系的对侧一个类和类名
- Metadata#inverse_foreign_key 返回对侧的外键名
- Metadata#inverse_klass 返回关系对侧的类
- Metadata#inverse_metadata 返回对侧关系的元数据
- Metadata#inverse_of 返回对侧关系的明确定义名
- Metadata#inverse_of? 返回对侧福安溪是否明有inverse_of选项
- Metadata#inverse_setter 返回逆关系的set方法名
- Metadata#inverse_type 返回逆关系的多态field名
- Metadata#inverse_type_setter 返回逆关系的多态field的setter
- Metadata#key  返回获取关系必须的属性名
- Metadata#klass 返回代理document的类名
- Metadata#macro 返回关系的宏
- Metadata#name 返回关系的名字
- Metadata#options 返回自身
- Metadata#order 返回关系选项的自定义的排序
- Metadata#order? 返回自定义排序是否设置
- Metadata#polymorphic? 返回关系是否多态
- Metadata#setter 返回设置关系的filed的setter方法
- Metadata#store_as 返回存储embedded关系的属性名
- Metadata#touchable? 返回关系是否有touch方法
- Metadata#type 返回获取多态关系filed的get方法.
- Metadata#type_setter 返回设置多态关系field的set方法
- Metadata#validate? 返回关系是否有关联校验
- Metadata#versioned? 返回关系是否有embedded版本
>

###Embedded 1-1
> 子内嵌在父document中 embeds_one 和 embedded_in,定义必须在双方都需要
	class Band
	  include Mongoid::Document
	  embeds_one :label
	end
	class Label
	  include Mongoid::Document
	  field :name, type: String
	  embedded_in :band
	end 
####存储
> 子document以hash形式存储在父document的collection中
	{
	  "_id" : ObjectId("4d3ed089fb60ab534684b7e9"),
	  "label" : {
	    "_id" : ObjectId("4d3ed089fb60ab534684b7e0"),
	    "name" : "Mute",
	  }
	}
> 可以设置内嵌document存储在名字以外的属性 使用:store_as选项
	class Band
	  include Mongoid::Document
	  embeds_one :label, store_as: "lab"
	end
####操作
> - Model#{name}
>
>	获取内嵌document
	band.label
> - Model#{name}=
>
>	设置内嵌文档, 父文档存储时自动存储子文档, 子文档设为nil则自动删除
	band.label = Label.new(name: "Mute")
	band.label = nil
> - Model#{parent_name}
>
>	从子文档中干获取父文档
	label.band
> - Model#{parent_name}=
>
>	设置子文档的父文档
	label.band = Band.new
> - Model#build_{name}
>
>	在关系上新建文档
	band.build_label(name: "Mute")
	label.build_band(name: "Depeche Mode")
> - Model#create_{name}
>
>	从关系的另一侧建立新文档, 父建子则子立即存储, 子建父则存储整个
	band.create_label(name: "Mute")
	label.create_band({
	  name: "Depeche Mode"
	})
###Embedded 1-n
> 1到多内嵌 embeds_many 和 embedded_in, 父和子文档中都要实现
	class Band
	  include Mongoid::Document
	  embeds_many :albums
	end
	class Album
	  innclude Mongoid::Document
	  field :name, type: String
	  embedded_in :band
	 end
####存储
> 子文档以数组形式存储在父文档的collection中
	{
	  "_id" : ObjectId("4d3ed089fb60ab534684b7e9"),
	  "albums" : [
	    {
	      "_id" : ObjectId("4d3ed089fb60ab534684b7e0"),
	      "name" : "Violator",
	    }
	  ]
	}
> 可以使用:store_as选项指定存储名
	class Band
	  include Mongoid::Document
	  embeds_many :albums, store_as: "albs"
	end
####操作
> - Model#{name}
>
>	获取子文档
	band.albums
> - Model#{name}=
>
>	设置子文档, 父文档存储是立即存储子文档, 设置为nil或\[\]则删除
	band.albums = [ Album.new(name: "Violator") ]
	band.albums = nil
	band.albums = []
> - Model#{parent_name}
>
>	从子文档中获取父文档
	album.band
> - Model#{parent_name}=
>
>	从子文档中设置父文档
	album.band = Band.new
> - Model#{name}.<<
> - Model#{name}.push
>
>	根据关系插入一个新文档, 父文档存储时, 子文档同时存储
	band.albums << Album.new(name: "Violator")
	band.albums.push(Album.new(name: "Violator"))
> - Model#{name}.concat
>
>	插入过个文档 , 父文档存储则子文档也存储
	band.albums.concat(
	  Album.new(name: "Violator"),
	  Album.new(name: "101")
	)
> - Model#{name}.build
> - Model#{name}.new
>
>	Build一个新文档 但不保存
	band.albums.build(name: "Violator")
	band.albums.new(name: "Violator")
> - Model#{name}.create
> - Model#{name}.create!
>
>	建立一个新文档并保存, 校验失败时Create!抛出错误
	band.albums.create(name: "Violator")
	band.albums.create!(name: "Violator")
> - Model#{name}.clear
> - Model#{name}.delete_all
>
>	删除所有文档, 不执行回调
	band.albums.clear
	band.albums.delete_all
	band.albums.delete_all(name: "Violator")
	band.albums.
	  where(name: "Violator").delete_all
> - Model#{name}.destroy_all
>
>	删除所有并执行回调
	band.albums.destroy_all
	band.albums.destroy_all(name: "Violator")
	band.albums.
	  where(name: "Violator").destroy_all
> - Model#{name}.delete
>
>	删除匹配文档
	band.albums.delete(album)
> - Model#{name}.pop
>
>	删除指定数目文档, 默认1个
	band.albums.pop
	band.albums.pop(1)
> - Model#{name}.find
>
>	返回匹配id的文档, 抛出错误如果找不到
	band.albums.find(id)
	band.albums.find(id_one, id_two)
> - Model#{name}.find_or_create_by
>
>	查找并创建
	band.albums.
	  find_or_create_by(name: "Violator")
> - Model#{name}.find_or_initialize_by
>
>	创建并初始化
	band.albums.
	  find_or_initialize_by(name: "Violator")
> - Model#{name}.where
>
>	查找,支持任何criteria方法
	band.albums.where(name: "Violator")
> - Model#{name}.exists?
>
>	查找是否存在子document
	band.albums.exists?
###Referenced 1-1
> 1-1引用关系 has_one 和 belongs_to，两侧都要定义除非 一侧的model是被内嵌的
	class Band
	  include Mongoid::Document
	  has_one :studio
	end
	class Studio
	  include Mongoid::Document
	  field :name, type: String
	  belongs_to :band
	end
####存储
> 父子文档独立存储, 但在子文档中存储父类的引用外键

	# The parent band document.
	{ "_id" : ObjectId("4d3ed089fb60ab534684b7e9") }
	# The child studio document.
	{
	  "_id" : ObjectId("4d3ed089fb60ab534684b7f1"),
	  "band_id" : ObjectId("4d3ed089fb60ab534684b7e9")
	}
####操作
> - Model#{name}
>
>	获取子文档
	band.studio
> - Model#{name}=
>
>	设置子文档. 父类存储时 子类也存储, 设置为nil后子文档被删除
	band.studio = Studio.new(name: "Abbey Road")
	band.studio = nil
> - Model#{parent_name}
>
>	从子中获取父名
	studio.band
> - Model#{parent_name}=
>
>	从子中设置父
	studio.band = Band.new
> - Model#build_{name}
>
>	建立新文档, 不保存
	band.build_studio(name: "Abbey Road")
	studio.build_band(name: "Depeche Mode")
> - Model#create_{name}
>
>	创建并保存, 如果是子创建则立即保存父, 父创建则立即保存子
	band.create_studio(name: "Abbey Road")
	studio.create_band(name: "Depeche Mode")
###Referenced 1-n
> 1到多引用关系 has_many 和 belongs_to, 两侧都要定义 除非一侧是内嵌的
	class Band
	  include Mongoid::Document
	  has_many :members
	end
	class Member
	  include Mongoid::Document
	  field :name, type: String
	  belongs_to :band
	end
####存储
> 父子独立, 子通过外键引用父
	# The parent band document.
	{ "_id" : ObjectId("4d3ed089fb60ab534684b7e9") }
	# The child member document.
	{
	  "_id" : ObjectId("4d3ed089fb60ab534684b7f1"),
	  "band_id" : ObjectId("4d3ed089fb60ab534684b7e9")
	}
	操作
> - Model#{name}
>
>	获取子文档
	band.members
> - Model#{name}=
>
>	设置子文档, 父存储则子立即存储 设为nil或g\[\]则删除
	band.members = [ Member.new(name: "Fletch") ]
	band.members = nil
	band.members = []
> - Model#{name}_ids
>
>	获取子文档id
	band.member_ids
> - Model#{name}_ids=
>
>	设置子文档ids
	band.member_ids = [ id ]
> - Model#{parent_name}
>
>	从子获取父
	member.band
> - Model#{parent_name}=
>
>	从子设置父
	member.band = Band.new
> - Model#{name}.<<
> - Model#{name}.push
>
>	插入新子文档, 如果父保存则子保存
	band.members << Member.new(name: "Fletch")
	band.members.push(Member.new(name: "Fletch"))
> - Model#{name}.concat
>
>	插入多个文档, 如果父保存则子批量保存
	band.members.concat(
	  Member.new(name: "Fletch"),
	  Member.new(name: "Martin")
	)
> - Model#{name}.build
> - Model#{name}.new
>
>	建立新文档,不保存
	band.members.build(name: "Fletch")
	band.members.new(name: "Fletch")
> - Model#{name}.create
> - Model#{name}.create!
>
>	建立并保存, create!校验失败抛出错误
	band.members.create(name: "Fletch")
	band.members.create!(name: "Fletch")
> - Model#{name}.clear
> - Model#{name}.delete_all
>
>	删除关系上的所有文档, 不调用回调
	band.members.clear
	band.members.delete_all
	band.members.delete_all(name: "Fletch")
	band.members.
	  where(name: "Fletch").delete_all
> - Model#{name}.destroy_all
>
>	删除所有文档并调用回调
	band.members.destroy_all
	band.members.destroy_all(name: "Fletch")
	band.members.
	  where(name: "Fletch").destroy_all
> - Model#{name}.delete
>
>	删除匹配文档
	band.members.delete(member)
> - Model#{name}.find
>
>	根据id查抄, 查找不到抛出错误
	band.members.find(id)
	band.members.find(id_one, id_two)
> - Model#{name}.find_or_create_by
>
>	查找, 找不到则创建
	band.members.
	  find_or_create_by(name: "Fletch")
> - Model#{name}.find_or_initialize_by
>
>	查找并初始化
	band.members.
	  find_or_initialize_by(name: "Fletch")
> - Model#{name}.where
>	
>	查找匹配文档, where外的任何criteria方法亦支持
	band.members.where(name: "Fletch")
> - Model#{name}.exists?
>
>	是否有文档存在
	band.members.exists?
###Referenced n-n
> 多到多关系 has_and_belongs_to_many, 两侧都存储外键
	class Band
	  include Mongoid::Document
	  has_and_belongs_to_many :tags
	end
	class Tag
	  include Mongoid::Document
	  field :name, type: String
	  has_and_belongs_to_many :bands
	end	
> 如果建立单侧的多到多引用, 可以模仿has_many 只在父中存储外键
	class Band
	  include Mongoid::Document
	  has_and_belongs_to_many :tags, inverse_of: nil
	end
	class Tag
	  include Mongoid::Document
	  field :name, type: String
	end
####存储
> 独立存储, 互相用外键关联
	# The band document.
	{
	  "_id" : ObjectId("4d3ed089fb60ab534684b7e9"),
	  "tag_ids" : [ ObjectId("4d3ed089fb60ab534684b7f2") ]
	}
	# The tag document.
	{
	  "_id" : ObjectId("4d3ed089fb60ab534684b7f2"),
	  "band_ids" : [ ObjectId("4d3ed089fb60ab534684b7e9") ]
	}
####操作
> 多到多操作通常需要双倍连接数据库来保持两侧关系同步, 通常会慢一些
>
> - Model#{name}
>
>	获取关联document
	band.tags
> - Model#{name}=
>
>	设置关联document, 父保存则子保存, 设置为nil或\[\]则删除
	band.tags = [ Tag.new(name: "electro") ]
	band.tags = nil
	band.tags = []
> - Model#{name}.<<
> - Model#{name}.push
>
>	添加一个新关联document, 父保存则子亦保存
	band.tags << Tag.new(name: "electro")
	band.tags.push(Tag.new(name: "electro"))
> - Model#{name}.concat
>
>	添加多个新关联document, 父保存则子亦保存
	band.tags.concat(
	  Tag.new(name: "electro"),
	  Tag.new(name: "new wave")
	)
> - Model#{name}.build
> - Model#{name}.new
>
>	创建但不保存
	band.tags.build(name: "electro")
	band.tags.new(name: "electro")
> - Model#{name}.create
> - Model#{name}.create!
>
>	创建并保存, create!会抛出错误
	band.tags.create(name: "electro")
	band.tags.create!(name: "electro")
> - Model#{name}.clear
> - Model#{name}.delete_all
>
>	删除所有, 不回调
	band.tags.clear
	band.tags.delete_all
	band.tags.delete_all(name: "electro")
	band.tags.
	  where(name: "electro").delete_all
> - Model#{name}.destroy_all
>
>	删除所有, 带回调
	band.tags.destroy_all
	band.tags.destroy_all(name: "electro")
	band.tags.
	  where(name: "electro").destroy_all
> - Model#{name}.delete
>
>	删除匹配document
	band.tags.delete(tag)
> - Model#{name}.find
>
>	查找 找不到抛出错误
	band.tags.find(id)
	band.tags.find(id_one, id_two)
> - Model#{name}.find_or_create_by
>
>	查找, 找不到创建一个
	band.tags.
	  find_or_create_by(name: "electro")
> - Model#{name}.find_or_initialize_by
>
>	查找, 找不到则初始化一个
	band.tags.
	  find_or_initialize_by(name: "electro")
> - Model#{name}.where
>
>	找到匹配的, 支持where外的所有criteria方法
	band.tags.where(name: "electro")
>- Model#{name}.exists?
>
>	是否有document存在
	band.tags.exists?