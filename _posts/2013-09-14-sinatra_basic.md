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

参考自[Sinatra Readme](http://www.sinatrarb.com/intro-zh.html)

基于Rack, DSL

### 路由

HTTP方法 + URL匹配, 每条路由关联到一个block, 前面的优先级高

| get	| - | show显示 |
| post | - | create创建 |
| put | - | replace更新(有就替换, 没有则新增, 一般要求包含所有属性资料) |
| patch | - | modify修改(部分更新) |
| delete | - | destroy删除 |
| options | - | appease功能选项, 查看服务器的性能,支持方法 |
| link | - | affiliate请求服务器建立链接关系 |
| unlink | - | separate请求服务器断开链接关系 |

#### 具名参数 :XXX

params[:XXX] 或 block参数

{% highlight ruby %}
get '/hello/:name' { ... }
{% endhighlight %}

#### 通配符 splat参数

params[:splat] 或 block参数

{% highlight ruby %}
get '/hello/*/to/*' { ... }
{% endhighlight %}

#### 正则表达式

params[:captures] 或 block参数

{% highlight ruby %}
get %r{/hello/([\w]+)} { ... }
{% endhighlight %}

匹配扩展名

{% highlight ruby %}
# 匹配 "GET /posts, /posts.json, /posts.xml" 等.
get '/posts.?:format?' { ... }
{% endhighlight %}

### 条件

路由支持条件匹配

| :agent	| - | 用户浏览器 |
| :host_name | - | |
| :provides | - | |

#### 自定义条件 

{% highlight ruby %}
set(:probability) { |value| condition { rand <= value } }

get '/win_a_car', :probability => 0.1 do
  "You won!"
end

get '/win_a_car' do
  "Sorry, you lost."
end
{% endhighlight %}

自定义条件中使用*来标识多参数匹配

{% highlight ruby %}
set(:auth) do |*roles|   # <= *参数
  condition do
    unless logged_in? && roles.any? {|role| current_user.in_role? role }
      redirect "/login/", 303
    end
  end
end

get "/my/account/", :auth => [:user, :admin] do
  "Your Account Details"
end

get "/only/admin/", :auth => :admin do
  "Only admins are allowed here!"
end
{% endhighlight %}

### 返回值

路由的block块的返回值确定返回给Http客户端的respond body(传递给Rack栈中的下一个中间件), 符合Rack协议

可以返回任何对象, 包括合理的Rack响应, Rack的body对象 或 HTTP状态码

- 包括三个元素的数组  [status (Fixnum), headers (Hash), response body (responds to #each)]
- 两个元素是数组  [status (Fixnum), response body (responds to #each)]
- 能响应#each方法, 每次产生一个字符串 
- 代表HTTP状态码的数字

### 静态文件

默认在./public, 可以通过设置:public_folder选项重设, 引用URL中不包含公共目录名

设置 :static_cache_control 增加Cache-Control头信息

{% highlight ruby %}
set :public_folder, File.dirname(__FILE__) + '/static'
{% endhighlight %}

### 视图/模板

每种模板语言有关联的rendering方法

- 这些rendering方法一般都返回字符串
- 接受的参数除了模板名(:XXX), 还可以是模板内容(字符串)
- 第二个参数是以选项hash, 选项还可以以set的方式设置到所有的rendering方法中

支持的选项

| locals | - | 传递参数 |
| default_encoding | - | 编码方法, 默认 settings.default_encoding |
| views | - | 模板路径, 默认 settings.views |
| layout | - | 是否使用layout(值为true或false), 使用哪个模板(值为符号) |
| content_type | - | 设置Content-Type, 默认值取决于模板语言 |
| scope | - | 渲染模板的范围, 默认是一共用实例, 修改后实例变量和辅助方法将失效 |
| layout_engine | - | 渲染layout的引擎 |
| layout_options | - | 适用于渲染layout |

模板默认在 ./view目录, 要使用不同的视图目录

{% highlight ruby %}
set :views, settings.root + '/templates'
{% endhighlight %}

使用符号渲染子目录下的模板 **:'subdir/template'** 或者 **'subdir/template'.to_sym**, 不能直接使用字符串

#### 文字模板

渲染字符串

#### 支持的模板语言

对于一种语言有多重实现工具的情况, 需要指定工具, 即先require

| 语言 | | 工具 |
|----
| Haml | - | haml|
| Erb | - | erubis, erb |
| Builder | - | builder |
| Nokogiri | - | nokogirl |
| Sass | - | sass |
| SCSS | - | scss |
| Less | - | less |
| Liquid | - | liquid |
| Markdown | - |  RDiscount, RedCarpet, BlueCloth, kramdown, maruku |
| Textile | - | RedCloth |
| RDoc | - | RDoc |
| Radius | - | Radius |
| Markaby | - | Markaby |
| RABL | - | RABL |
| Slim | - | Slim |
| Creole | - | Creole |
| CoffeeScript | - | CoffeeScript |
| Stylus | - | Stylus |
| Yajl | - | Yajl |
| WLang | - | WLang |
{: rules="groups"}

#### 模板中访问变量 

模板和路由句柄有相同的上下文, 路由句柄中设置的变量可以在模板中直接使用, 还可以使用:local传递新嵌套

#### yield和layout嵌套

layout是一个包含yield的模板, 通过传递block给rendering方法实现layout嵌套

{% highlight ruby %}
erb :main_layout, :layout => false do
  erb :admin_layout do
    erb :user
  end
end
{% endhighlight %}

#### 内联模板

在源文件末尾定义模板

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

#### 具名模板

使用顶层的 template 方法定义

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

#### 关联文件名

关联文件扩展名到模板引擎 Tilt.register

{% highlight ruby %}
Tilt.register :tt, Tilt[:textile]
{% endhighlight %}

#### 自定义模板引擎

{% highlight ruby %}
Tilt.register :myat, MyAwesomeTemplateEngine

helpers do
  def myat(*args) render(:myat, *args) end
end

get '/' do
  myat :index
end
{% endhighlight %}

### 过滤器

befor过滤器在同一上下文的每个request之前执行, 可以修改request和response, 设置的实例变量可以被路由, 模板, after过滤器访问

after过滤器在同一上下文的每个request之后执行

除非是在route方法中返回了字符串, 否则早after过滤器中不能使用body

过滤器支持凡是匹配 和 条件

{% highlight ruby %}
before '/protected/*' do
  authenticate!
end

after '/create/:slug' do |slug|
  session[:last_slug] = slug
end
{% endhighlight %}

{% highlight ruby %}
before :agent => /Songbird/ do
  # ...
end

after '/blog/*', :host_name => 'example.com' do
  # ...
end
{% endhighlight %}

### 辅助方法

使用顶层的helpers方法来定义辅助方法, 以便在路由handler和模板中使用

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

在请求之间保持状态, 每一个用户会话对应一个session hash

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

#### 挂起 halt

直接停止请求, 在过滤器或路由中使用 **halt**

支持指定状态码(数字), 消息体(字符串), 消息头(Hash)

{% highlight ruby %}
halt 402, {'Content-Type' => 'text/plain'}, 'revenge'
{% endhighlight %}

#### 让路 pass

一个路由放弃处理, 路由block直接退出, 将处理让个下一个匹配路由 pass

#### 触发另一个路由 call

{% highlight ruby %}
get '/foo' do
  status, headers, body = call env.merge("PATH_INFO" => '/bar')
  [status, headers, body.map(&:upcase)]
end

get '/bar' do
  "bar"
end
{% endhighlight %}

请求发给同一个应用 而不是副本, 使用call!

#### 设定消息体, 状态码和消息头

使用辅助方法 status headers body

{% highlight ruby %}
get '/foo' do
  status 418
  headers \
    "Allow"   => "BREW, POST, GET, PROPFIND, WHEN",
    "Refresh" => "Refresh: 20; http://www.ietf.org/rfc/rfc2324.txt"
  body "I'm a tea pot!"
end
{% endhighlight %}

#### 流响应

stream辅助方法

{% highlight ruby %}
get '/' do
  stream do |out|
    out << "It's gonna be legen -\n"
    sleep 0.5
    out << " (wait for it) \n"
    sleep 1
    out << "- dary!\n"
  end
end
{% endhighlight %}

需要web server支持

keep_open 参数, 只在Event服务器上 如Thin才生效

#### 日志

logger方法指向Logger实例, 如果logging不使能则返回虚拟的对象

默认在Sinatra::Application中enable, 在Sinatra::Base中需要手动enable

#### 媒体类型

使用 mime_type注册后, 使用content_type

{% highlight ruby %}
configure do
  mime_type :foo, 'text/foo'
end
{% endhighlight %}

{% highlight ruby %}
get '/' do
  content_type :foo
  "foo foo foo"
end
{% endhighlight %}


#### 生成URL

使用url(别名to)辅助方法, 使用Rack路由, 代理自动计算url

#### 重定向 redirect

使用redirect方法触发浏览器重定向

类似halt方法 支持状态码 消息体等参数

redirect back支持用户返回

如果需要在redirect中传递参数, 可以在url中添加query参数, 或使用session

#### 缓存控制

设置 cache_control

{% highlight ruby %}
cache_control :public, :must_revalidate, :max_age => 60
{% endhighlight %}

使用 expires方法会自动设置对应的Cache-Control;

使用etag, last_modifield辅助方法来使用缓存

使用缓存的解决方法, 外挂Rack的缓存机制如rack-cache

#### 发送文件 send_file

支持的选项

| filename | - | 响应中的文件名，默认是真实文件的名字 |
| last_modified | - | 消息头Last-Modified的值，默认是文件的mtime（修改时间） |
| type | - | 使用的内容类型，如果没有会从文件扩展名猜测 |
| disposition | - | 用于 Content-Disposition，可能的包括： nil (默认), :attachment 和 :inline |
| length | - | Content-Length 的值，默认是文件的大小 |
| status | - | 常用与发送静态文件作为错误页面 | 

#### 请求对象 request

访问传入的请求

{% highlight ruby %}
# 在 http://example.com/example 上运行的应用
get '/foo' do
  request.accept              # ['text/html', '*/*']
  request.accept? 'text/xml'  # true
  request.preferred_type(t)   # 'text/html'
  request.body              # 被客户端设定的请求体（见下）
  request.scheme            # "http"
  request.script_name       # "/example"
  request.path_info         # "/foo"
  request.port              # 80
  request.request_method    # "GET"
  request.query_string      # ""
  request.content_length    # request.body的长度
  request.media_type        # request.body的媒体类型
  request.host              # "example.com"
  request.get?              # true (其他动词也具有类似方法)
  request.form_data?        # false
  request["SOME_HEADER"]    # SOME_HEADER header的值
  request.referrer          # 客户端的referrer 或者 '/'
  request.user_agent        # user agent (被 :agent 条件使用)
  request.cookies           # 浏览器 cookies 哈希
  request.xhr?              # 这是否是ajax请求？
  request.url               # "http://example.com/example/foo"
  request.path              # "/example/foo"
  request.ip                # 客户端IP地址
  request.secure?           # false（如果是ssl则为true）
  request.forwarded?        # true （如果是运行在反向代理之后）
  request.env               # Rack中使用的未处理的env哈希
end
{% endhighlight %}

部分参数如 script_name or path_info 可以被复写

request.body 是一个IO对象或者 StringIO对象

#### 附件 attachment

通知浏览器 应该写入磁盘而不是在浏览器中显示

#### 处理时间日期

time_for方法 从指定参数生成Time对象 或转换DateTime Date

expires last_modified akin方法内置使用time_for

#### 查找模板

find_template 硬盘能够与在render是查找模板, 不常用

### 配置

application级别, 使用 set enable disable 方法, 支持 Proc

{% highlight ruby %}
configure :XXX do
  # setting one option
  set :option, 'value'

  # setting multiple options
  set :a => 1, :b => 2

  # same as `set :option, true`
  enable :option

  # same as `set :option, false`
  disable :option

  # you can also have dynamic settings with blocks
  set(:css_dir) { File.join(views, 'css') }
end
{% endhighlight %}

使用settings获取配置

#### 配置保护

使用Rack::Protection防护常见的攻击

| 关闭 | - | disable :protection |
| 跳过某一层防护 | - | set :protection, :except => [:path_traversal, :session_hijacking] |

默认session使能保护, 使用其他session手动打开

{% highlight ruby %}
use Rack::Session::Pool
set :protection, :session => true
{% endhighlight %}

#### 支持的配置项

absolute_redirects

	如果被禁用，Sinatra会允许使用相对路径重定向, 如果应用运行在一个未恰当设置的反向代理之后， 你需要启用这个选项。
	注意 url 辅助方法 仍然会生成绝对 URL，除非你传入 false 作为第二参数。
	默认禁用。

add_charsets

	设定 content_type 辅助方法会 自动加上字符集信息的多媒体类型。
	你应该添加而不是覆盖这个选项: settings.add_charsets << "application/foobar"

app_file

	主应用文件，用来检测项目的根路径，用来计算默认的:root :public_folder :views
	set :app_file, __FILE__

bind

	绑定的IP 地址 (开发环境下默认: 0.0.0.0)。 仅对于内置的服务器有用。

default_encoding

	默认编码 (默认为 "utf-8")。

dump_errors

	在log中显示错误。

environment

	当前环境，默认是 ENV['RACK_ENV']， 不可用则为 "development" 。

logging

	使用logger, 默认STRERR 可以使用Rack::CommonLogger

lock

	对每一个请求放置一个锁， 只使用进程并发处理请求。
	如果你的应用不是线程安全则需启动。 默认禁用。

method_override

	使用 _method 魔法以允许在旧的浏览器中在 表单中使用 put/delete 方法

port

	监听的端口号。只对内置服务器有用。

prefixed_redirects

	没有设定绝对路径时是否添加 request.script_name 到 重定向请求。那样的话 redirect '/foo' 会和 redirect to('/foo')起相同作用。默认禁用。

protection

	是否打开web攻击保护

public_folder, public_dir

	public文件夹的位置
	默认时root下的public目录, 通常设置为root的相对路径 set :public_folder, Proc.new { File.join(root, "static") }

reload_templates

	是否每个请求都重新载入模板。 在development mode启用

root

	项目的根目录。默认是包含主文件的目录, 常用来设置默认的:public_folder和:views
	在主文件中直接设置 set :root, File.dirname(__FILE__)

raise_errors

	抛出异常（应用会停下）。
	关闭后, 异常会rescued并映射到错误处理, 通常是返回5xx的状态码
	打开后, 异常被跑出到应用外, 可以由服务器或Rack中间件处理, 如Rack::ShowExceptions或Rack::MailExceptions

run

	如果启用，Sinatra会开启内置web服务器。 如果使用rackup或其他方式则不要启用。
	默认值 检查 :app_file是否和$0匹配
  
running

	内置的服务器在运行吗？ 不要修改这个设置！

server

	服务器，或用于内置服务器的列表。 默认是 [‘thin’, ‘mongrel’, ‘webrick’], 顺序表明了 优先级。

sessions

	开启基于cookie的sesson。

show_exceptions

	在浏览器中显示一个stack trace。默认在development环境中打开

static

	Sinatra是否处理静态文件(public目录下)。 当服务器能够处理则禁用。 禁用会增强性能。 默认开启。

static_cache_control

	设置Cache-Control, 默认关闭

threaded

	设置true, Thin将使用 EventMachine.defer处理request

views

	views 文件夹。设置同public_folder

x_cascade

	设置X-Cascade, 默认开启

### 环境

"development", "production" and "test"

### 错误处理

not_found

error

### Rack中间件

sinatra依靠Rack, 通过顶层use方法建立中间件的管道

### 测试

支持任何基于Rack的测试程序库或者框架, 推荐Rack::Test

### Sinatra::Base

