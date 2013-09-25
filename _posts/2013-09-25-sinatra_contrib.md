---
layout: post
title: sinatra_contrib扩展
description: "sinatra, contrib"
modified: 2013-09-25
tags: [sinatra, web]
image:
  feature: abstract-2.jpg
comments: true
share: true
---

参考自[Sinatra Contrib](http://www.sinatrarb.com/contrib/)

常用的sinatra扩展, 包括common和custom扩展两部分

common部分不修改已有API的行为, 不依赖其他Sinatra::Contrib, custom部分则相反

### 引用

单个扩展

{% highlight ruby %}
require 'sinatra/base'
require 'sinatra/content_for'
require 'sinatra/namespace'

class MyApp < Sinatra::Base
  helpers Sinatra::ContentFor
  register Sinatra::Namespace
end
{% endhighlight %}

comm扩展

{% highlight ruby %}
require 'sinatra/base'
require 'sinatra/contrib'

class MyApp < Sinatra::Base
  register Sinatra::Contrib
end
{% endhighlight %}

全部扩展

{% highlight ruby %}
require 'sinatra/base'
require 'sinatra/contrib/all'

class MyApp < Sinatra::Base
  register Sinatra::Contrib
end
{% endhighlight %}

### Common扩展

#### Sinatra::ConfigFile

yaml格式配置文件, 自动检测文件是否包含明确的环境设置并使用 **register**

{% highlight ruby %}
require "sinatra/base"
require "sinatra/config_file"

class MyApp < Sinatra::Base
  register Sinatra::ConfigFile

  config_file 'path/to/config.yml'

  get '/' do
    @greeting = settings.greeting
    haml :index
  end

  # The rest of your modular application code goes here...
end
{% endhighlight %}

#### Sinatra::ContentFor

Rails风格 content_for, 支持 Erb, Erubis, Haml and Slim. **helpers**

#### Sinatra::Cookies

提供cookies辅助方法读写cookies. **helpers**

读

    cookies[:something]

写

    cookies[:something] = 'somevalue'

其他方法

     cookies.merge! 'foo' => 'bar', 'bar' => 'baz'
     cookies.keep_if { |key, value| key.start_with? 'b' }
     foo, bar = cookies.values_at 'foo', 'bar'
     cookies.length

#### Sinatra::JSON

提供json辅助方法, 返回JSON. **helpers**

{% highlight ruby %}
require "sinatra/base"
require "sinatra/json"

class MyApp < Sinatra::Base
  helpers Sinatra::JSON

  # define a route that uses the helper
  get '/' do
    json :foo => 'bar'
  end

  # The rest of your modular application code goes here...
end
{% endhighlight %}

设置默认编码器, content_type

{% highlight ruby %}
set :json_encoder, XXXX
set :json_content_type, :js
{% endhighlight %}

也可以在json方法中传入 hash设置

{% highlight ruby %}
get '/'  do
  json({:foo => 'bar'}, :encoder => :to_json, :content_type => :js)
end
{% endhighlight %}

#### Sinatra::LinkHeader

提供辅助方法, 生成link HTML标签和相应的 Link HTTP头 **helpers**

- prefetch 
- stylesheet
- link
- link_headers

#### Sinatra::MultiRoute

1个声明对应多路由 **register**

多个HTTP动词 + 多个路由, 支持自定义动词

{% highlight ruby %}
route :get, :post, ['/foo', '/bar'] do
  # ...
end
{% endhighlight %}

#### Sinatra::Namespace

支持namespaces **register**

URL分级, namespace可以嵌套

#### Sinatra::RespondWith

根据请求的Accept头字段, 选择action和模板 **register**

respond_to 限制返回类型

respond_with

#### Sinatra::Streaming

仿造IO对象创建stream对象来增强streaming API **helpers**

### Custom 扩展

#### Sinatra::Decompile

重建路径范式 **register**

提供decompile方法, 对给定的路由生成一个范式

#### Sinatra::Reloader

文件变化时, 自动重加载ruby **register**

{% highlight ruby %}
require "sinatra/base"
require "sinatra/reloader"

class MyApp < Sinatra::Base
  configure :development do
    register Sinatra::Reloader
  end

  # Your modular application code goes here...
end
{% endhighlight %}

also_reload 加载其他
dont_reload 不加载

### 其他工具

#### Sinatra::Extension

为新增sinatra扩展提供Mixin
