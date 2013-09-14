---
layout: post
title: Rack基础
description: "rack学习笔记"
modified: 2013-09-14
tags: [sinatra, web]
image:
  feature: abstract-8.jpg
comments: true
share: true
---

[API 文档](http://rack.rubyforge.org/doc/)

[官方主页](http://rack.github.io/)

### Rack

Ruby服务器(_Thin_,_Unicorn_,_WEbrick_)和Rack应用程序(_Rails_,_Merb_,_Sinatra_)之间的接口

#### 标准接口

Rack 通过handler机制实现对应用服务器的支持

##### Rack Handler

Rack 内置了[:CGI, :FastCGI, :Mongrel, :EventedMongrel, :SwiftipliedMongrel, :WEBrick, :LSWS, :SCGI, :Thin]的Hander, 其他Web框架也在其代码中包含一个Rack是配置, 调用对应的run方法就可以运行Rack程序, 如

{% highlight ruby %}
Rack::Handler:: Thin.run ...
{% endhighlight %}

##### Rack程序

- 一个ruby对象
- 能够相应call
- 接受一个参数, 返回数组, 包含 status headers body三个值
  - status 不一定必须是整数, 必须能够响应to_i方法返回一个整数, 整数必须>100
  - headers 必须能够响应each方法, 每次产生一个key和value, key和value必须是字符串
    在状态码是1xx,204, 304时,必须含有Content-Type和Content-length
  - body 必须能够响应each, 每次必须产生一个字符串, 即字符串数组

ruby中能够被call的对象: lambda, proc, method, 任何包含一个call方法的对象

最简单的空lambda对象

{% highlight ruby %}
Rack::Handler::WEBrick.run lambda{|env| [200, {}, ["hello from lambda"]]}, :Port=>3000
{% endhighlight %}

### 环境

{% highlight ruby %}
Rack::Handler::WEBrick.run lambda{|env| [200, {}, [env.to_json]]}, :Port=>3000
{% endhighlight %}

#### 大写的CGI头类变量

几个重要的Key

- REQUEST_METHOD HTTP请求的方法，可以是GET, POST等等。
- PATH_INFO 程序所要处理的“路径”，利用它我们可以实现不同的“路由”算法。
- QUERY_STRING URL中的?后面的一部分, 参数

程序中可以通过 env变量获取这些信息

{% highlight ruby %}
env['REQUEST_METHOD']
{% endhighlight %}

#### rack.xxxx

rack要求环境中必须包含的一些变量

### Request

创建一个Request对象

{% highlight ruby %}
request = Rack::Request.new(env)
{% endhighlight %}

新建的request对象直接持有传入的env对象,并能在需要的时候修改它

可以直接以hash形式获取用户的请求参数

{% highlight ruby %}
request.params[somekey]
{% endhighlight %}

Request提供了询问当前HTTP请求类型的简单方法

- request_method() 请求的HTTP方法, 包括 GET POST PUT DELETE HEAD
- get?() head?() post?() put?() delete?() HTTP请求是否为 GET HEAD POST PUT DELETE
- xhr?() HTTP请求是否为XMLHttpRequest(即Ajax请求)

### Response

提供对相应的状态, HTTP头和内容进行处理的方便接口

#### 响应体

两种方法
1. 直接设置response.body 需手工设置响应头中的Content-Length
2. 使用response.write增量写入内容, 自动填充Content-Length

最后使用response.finish完成, finish将装配除符合Rack规范的数组


#### 状态码

直接存取Response的对象来改变状态码, 默认为200

{% highlight ruby %}
response.status = 200
{% endhighlight %}

redirect方法直接重定向

{% highlight ruby %}
redirect(target, status=302)
{% endhighlight %}


#### 响应头

直接通过Hash写入响应头

{% highlight ruby %}
response.headers['Content-Type'] = 'text/plain'
{% endhighlight %}

### 中间件

ruby应用服务器和Rack应用程序之间的代码

中间件必须是一个合法的Rack应用程序

中间件=>通用逻辑和业务逻辑分离

#### 装配中间件

在一个应用程序中使用多个中间件

- 直接使用 new方法

{% highlight ruby %}
Rack::Handler::XXXX.run Middleware1.new(Middleware2.new(rack_app))
{% endhighlight %}

- DSL 定义一个类和几个方法, 方法成为DSL中的动词

{% highlight ruby %}
Builder.new {
  use Middleware1
  use Middleware2
  run Rack Application
}
{% endhighlight %}

{% highlight ruby %}
app =Builder.new {
  use Rack::ContentLength
  use Decorator
  run lambda {|env| [200, {"Content-Type"=>"text/html"}, ["hello world"]]}
}.to_app

Rack::Handler::WEBrick.run app, :Port => 3000
{% endhighlight %}

##### Builder实现

Rack::Builder 粘合多种Rack中间件

- initialize 它的签名应该是initialize(&block)，为了能够让use、run这些方法成为DSL语言的动词，initialize应该instance_eval当前实例。
- use 它的签名应该是use(middlewareclass, options)，它应该记录需要创建的中间件以及它的顺序。
- run 它的签名应该是run (rack_app)，它应该纪录原始的rack应用程序
- to_app 根据use和run记录的信息创建出最终的应用程序

lambda实现

{% highlight ruby %}
class Builder
  def initialize(&block)
   @middlewares = []
   self.instance_eval(&block)
  end

  def use(middleware_class,*options, &block)
    @middlewares << lambda {|app| middleware_class.new(app,*options, &block)}
  end
  
  def run(app)
    @app = app
  end
  
  def to_app
    @middlewares.reverse.inject(@app) { |app, middleware| middleware.call(app)}
  end
end
{% endhighlight %}

### Rackup

简单的 web框架

- 存取Request Response
- 路由
- cookie session
- 生成日志

Rack::Builder

Rack的路由 

{% highlight ruby %}
map 'XXX' do
  run ...
end
{% endhighlight %}

####Rackup

提供Rackup命令, 用一个配置文件运行程序 config.ru, 使用Rack::Builder配置中间件

Rack::Server

### 中间件

####HTTTP协议中间件

##### Rack::Chunked

分块传输编码

##### Rack::ConditionGet

缓存机制

##### Rack::ContentLength

必要时使用一次,通常在最外层use

##### Rack::ContentType

##### Rack::Deflater

##### Rack::Etag

##### Rack::Head

##### Rack::MethodOverride

浏览器和Web服务器一般不直接支持PUT和DELETE方法, 使用POST方法模拟之

1. 在POST提交的表单数据中嵌入一个hidden字段区分这是什么方法 _method=put
2. HTTP请求中加一个扩展的X_HttP_METHOD_OVERRIDE请求头

#### 程序开发中间件

##### Rack::CommonLogger

纪录日志信息, 使用Apache common log 格式化把每一个请求的信息纪录到一个logger中

logger必须是合法的错误流ErrorStream, 要符合

- 能响应 puts write flush方法
- 可以用给一个参数调用puts, 只要这个参数能响应to_s
- 可以用给一个参数调用write, 只要这个参数是String
- 可以无参数的调用flush, 从而保证日志信息正确写入

如 Rack::CommonLogger $stderr

不设置则从环境变量的rack.errors中获取对应值

##### Rack::Lint

检查请求和响应是否符合Rack规格, 静态检查 call, 动态检查 each

检查不通过则爬出Rack::lint::LintError

##### Rack:Reloader

载入修改而不重启Web服务

##### Rack::Runtime

计算请求的处理时间

##### Rack::Sendfile

#### 应用配置和组合中间件

##### Rack::Cascade

挂载锁个应用程序, 请求来到时 尝试所有的应用程序直到某一个返回的代码不是404

##### Rack::Lock

多线程

#### 会话管理

服务端能够跟踪用户并和它多次交互, 首先要确定用户身份, Cookie

##### Http Cookie

用户第一次访问Web, 服务器端不指导用户的任何信息, 为确认其后续访问, 在服务端设置一个唯一的cookie来标识用户, 然后在响应头中设置 Set-Cookie字段值为此cookie

浏览器保存此cookie,下次用户访问同一个网站时, 浏览器选取对应的cookie, 用它设置请求头中的Cookie字段

Cookie中可以包含多个key-value值对

通常cookie含有

- domain cookie域名, 可选, 浏览器根据此判断是否要把cookie发送给某一域名的主机
- path cookie相关域名上的开始路径, 可选, 让cookie与网站的一部分关联
- secure 是否安全, 可选, 设置后只有在https才发送此cookie
- expires 过期时间, 设置与否决定cookie是会话cookie还是持久cookie, 格式 weekday, DD-Mon-YY HH::MM::SS GMT
- 
	ruby中使用 Time.now.gmtime.strftime("%a, %d-%b-%Y %H:%M:%S GMT")生成
- name cookie名字
- value cookie值







 




