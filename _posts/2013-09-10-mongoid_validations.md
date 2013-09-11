---
layout: post
title: "mongoid_9-校验"
description: "mongoid的字段校验"
category: mongoid
tags: [mongoid]
---

参考自[Validations](http://mongoid.org/en/mongoid/docs/validation.html)

> mongoid引用ActiveModel::Validations支持校验

###公共选项

> 所有校验器支持

	:allow_nil 是否允许空属性
	:if 提供的值是true.
	:on 只在指定情况下, 支持 :create 和 :update.
	:unless 提供的值是false

###校验项

> - 是否验收

	validates_acceptance_of :terms
	validates :terms, acceptance: true
	
> > 可选项

	:message
	:accept # 指定验收时接受时的值

> - 校验父是否有效

	validates_associated :albums
	validates :albums, associated: true

> > 可选项

	:message

> - 是否确认, 使用_confirmation后缀标示确认field

	validates_confirmation_of :password
	validates :password, confirmation: true

> > 可选项

	:message

> - 校验是否列表中内容

	validates_exclusion_of :employers, in: [ "SoundCloud" ]
	validates :employers, exclusion: { in: [ "SoundCloud" ] }

> > 可选项

	:in # Required list of values.
	:message

> - 校验格式

	validates_format_of :title, with: /\A\w+\Z/
	validates :title, format: { with: /\A\w+\Z/ }

> > 可选项

	:allow_blank # 能否为空
	:in # 值列表或范围
	:message
	:with # 匹配的正则表达式
	:without # 不匹配的正则表达式

> - 校验是否是列表中值

	validates_inclusion_of :employers, in: [ "SoundCloud" ]
	validates :employers, inclusion: { in: [ "SoundCloud" ] }

> > 可选项

	:allow_blank # 能否为空
	:in # 值列表.
	:message

> - 校验长度

	validates_length_of :password, minimum: 8, maximum: 16
	validates :password, length: { minimum: 8, maximum: 16 }

> > 可选项

	:allow_blank # 能否为空
	:in # 长度范围
	:maximum #最大长度
	:message
	:minimum # 最短长度
	:tokenizer # 块 tokenizer.
	:too_long # 过长提示信息
	:too_short # 过短提示信息
	:within # 范围.
	:wrong_length # 错误长度提示信息

> - 校验是否数据

	validates_numericality_of :age, even: true
	validates :age, numericality: { even: true }

> > 可选项

	:equal_to #必须相等的值
	:even
	:greater_than
	:greater_than_or_equal_to
	:less_than
	:less_than_or_equal_to
	:message
	:odd # 必须是奇数
	:only_integer # 必须是整数

> - 校验是否必须项
>
>	 绑定到关系上, 这个关系会自动保存

	validates_presence_of :name
	validates :name, presence: true

> > 可选项

	:message

> - 校验是否唯一

	embedded关系中只在父文档下校验不会校验整个数据库
	validates_uniqueness_of :name
	validates :name, uniqueness: true

> > 可选项

	:message
	:case_sensitive # 是否大小写匹配
	:scope # 值的范围