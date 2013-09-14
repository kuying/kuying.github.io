---
layout: post
title: sinatra基础
description: "初学sinatra笔记"
modified: 2013-09-14
tags: [sinatra, web]
image:
  feature: abstract-8.jpg
comments: true
share: true
---

参考自[Sinatra](http://www.sinatrarb.com/intro-zh.html)

基于Rack

DSL

### 路由

具名参数

{% highlight ruby %}
get '/hello/:name' ...
{% endhighlight %}

通配符

{% highlight ruby %}
get '/hello/*/to/*' ...
{% endhighlight %}

正则表达式

{% highlight ruby %}
get %r{/hello/([\w]+)} ...
{% endhighlight %}

条件匹配

:agent :host_name :provides

自定义条件 

{% highlight ruby %}
set(:probability) { |value| condition { rand <= value } }

get '/win_a_car', :probability => 0.1 do
  "You won!"
end

get '/win_a_car' do
  "Sorry, you lost."
end
{% endhighlight %}

自定义路由匹配

{% highlight ruby %}
class AllButPattern
  Match = Struct.new(:captures)

  def initialize(except)
    @except   = except
    @captures = Match.new([])
  end

  def match(str)
    @captures unless @except === str
  end
end

def all_but(pattern)
  AllButPattern.new(pattern)
end

get all_but("/index") do
  # ...
end
{% endhighlight %}

#### params参数

路由范式包含具名参数 :XXXX 可以通过params[:XXXX]获得

包含通配符的参数, 通过params[:splat]获得, splat参数中不包含正则表达式匹配的翔安书

正则表达式匹配的路由, 通过 params[:capture] 获得

可使用块参数获得

#### 返回值

决定返回给Http客户端的respond

#### 静态文件

从 ./public_folder 提供服务, 可以通过设置:public选项设定不同位置

引用url中不包含public目录名

{% highlight ruby %}
set :public_folder, File.dirname(__FILE__) + '/static'
{% endhighlight %}

### 视图/模板

直接位于 ./view目录, 要使用不同的视图目录

{% highlight ruby %}
set :views, File.dirname(__FILE__) + '/templates'
{% endhighlight %}

可以通过符号引用模板, 即使在子目录下 **:'subdir/template'**, 不使用符号,则会直接作为字符串渲染

支持的模板 haml erb erubis builder nokogirl sass scss less luquid markdown textile rdoc radius markaby slim creole coffeescript 

模板中访问变量 

内联模板

在文件末尾定义

{% highlight ruby %}
require 'sinatra'

get '/' do
  haml :index
end

__END__

@@ layout
%html
  = yield

@@ index
%div.title Hello world!!!!!
{% endhighlight %}


具名模板

顶层template 方法定义

{% highlight ruby %}
template :layout do
  "%html\n  =yield\n"
end

template :index do
  '%div.title Hello World!'
end

get '/' do
  haml :index
end
{% endhighlight %}

关联文件名扩展

### 过滤器

befor after

支持范式 只有满足该范式才会执行, 支持条件

### 辅助方法

使用顶层的helpers方法来定义辅助方法, 以便在路由处理器和模板中使用

{% highlight ruby %}
helpers do
  def bar(name)
    "#{name}bar"
  end
end

get '/:name' do
  bar(params[:name])
end
{% endhighlight %}

#### sessions

在请求之间保持状态, 每一个用户会话对饮更有一个session hash

{% highlight ruby %}
enable :sessions

get '/' do
  "value = " << session[:value].inspect
end

get '/:value' do
  session[:value] = params[:value]
end
{% endhighlight %}

enable:session 保存所有数据在一个cookie中, 可以使用任何的Rack Session中间件 替代之

{% highlight ruby %}
use Rack::Session::Pool, :expire_after => 2592000

get '/' do
  "value = " << session[:value].inspect
end

get '/:value' do
  session[:value] = params[:value]
end
{% endhighlight %}

#### 挂起 halt

直接停止请求, 在过滤器或路由中使用 **halt**

支持指定状态码或消息体, 可以带消息头

{% highlight ruby %}
halt 402, {'Content-Type' => 'text/plain'}, 'revenge'
{% endhighlight %}

#### 让路 pass

一个路由放弃处理, 将处理让个下一个匹配路由 pass

{% highlight ruby %}
get '/guess/:who' do
  pass unless params[:who] == 'Frank'
  'You got me!'
end

get '/guess/*' do
  'You missed!'
end
{% endhighlight %}

####触发 call

{% highlight ruby %}
get '/foo' do
  status, headers, body = call env.merge("PATH_INFO" => '/bar')
  [status, headers, body.map(&:upcase)]
end

get '/bar' do
  "bar"
end
{% endhighlight %}

请求发给统一给应用 而不是副本, 使用call!

#### 设定消息体, 状态码和消息头

{% highlight ruby %}
get '/foo' do
  status 418
  headers \
    "Allow"   => "BREW, POST, GET, PROPFIND, WHEN",
    "Refresh" => "Refresh: 20; http://www.ietf.org/rfc/rfc2324.txt"
  body "I'm a tea pot!"
end
{% endhighlight %}

将会被Rack处理器执行

设置完后, 可以使用 body headers status访问他们的当前值, 如在 after中

#### 媒体类型

使用send_file或静态文件时, 通过mime_type: 扩展注册

也可以使用content_type方法

#### 生成URL

使用url辅助方法, 如在haml中 **%a{:href => url('/foo')} foo** 会自动计算url, 别名 to

#### 浏览器重定向 redirect

#### 缓存控制

#### 发送文件 send_file

#### 请求对象 request

#### 附件 attachment

通知浏览器 应该写入磁盘而不是在浏览器中显示

#### 查找模板

### 配置

### 错误

### Rack中间件

sinatra依靠Rack, 一个面向Ruby Web框架的最小标准接口

### 测试

支持任何基于Rack的测试程序库或者框架, 推荐Rack::Test

