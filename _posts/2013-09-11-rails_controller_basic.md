---
layout: post
title: "rails_3-控制器基础"
description: "rails控制器"
category: rails
tags: [rails]
---


HTTP是一种Request-Response(请求-回应)的流程。Rails使用路由来分派request到Controller的其中一个Action。Action根据客户端传来的数据与Model互动，然后回应结果给客户端。

###ApplicationController
***

默认所有的Controller都继承自ApplicationController。因此在这里定义的方法可以被所有Controller调用，可以在这边定义一些共用的方法。
>原生的application_controller.rb：

	class ApplicationController < ActionController::Base
	  protect_from_forgery
	end

其中的protect_from_forgery方法启动了CSRF安全性功能，所有非GET的HTTP request都必须带有一个Token参数才能存取，Rails会自动在所有表单中插入Token参数，预设的Layout中也有一行<%= csrf_meta_tag %>标籤可以让JavaScript读取到这个Token。会需要关闭这个功能的时机是，你需要开放API给非浏览器客户端，这时候你会需要取消它：

	class ApisController < ApplicationController
	  skip_before_filter :verify_authenticity_token
	end

除了继承自ApplicationController，我们也可以继承更底层的ActionController::Metal，只需要controller的部分功能，提高相应速度

	# lib/static_controller.rb
	class StaticController < ActionController::Metal
	  include ActionController::Rendering
    
	  append_view_path "#{Rails.root}/app/views"
    
	  def about
		render "about"
	  end
	end
    
	# config/route.rb
	match '/about', :to => "static#about", :as => :about

###Action
***
一个Action就是Controller的一个Public方法：

	class EventsController < ApplicationController
	  def show
		# ...
	  end
	end

####Action操作

1. 收集request的数据，例如传进来的参数
2. 操作Model
3. 回传response结果，这个动作称作render
>

#####处理request

***

- action_name 当前的Action名称
- cookies Cookie
- headers HTTP头
- params 用户所有传入参数Hash

	这个Hash是ActiveSupport::HashWithIndifferentAccess对象，Ruby内建Hash，使用符号hash\[:foo\]和字符串hash\["foo"\]不一样，会引起bug
- request 各种关于此request的详细资讯
> - request_method
> - method
> - delete?, get?, head?, post?, put?
> - xml_http_request? 或 xhr?
> - url
> - protocol, host, port, path 和 query_string
> - domain
> - host_with_port
> - port_string
> - ssl?
> - remote_ip?
> - path_without_extension, path_without_format_and_extension, format_and_extension, relative_path
> - env
> - accepts
> - format
> - mime_type
> - content_type
> - headers
> - body
> - content_length
- response 代表要回传的内容，会由Rails设定好。通常你会用到的时机是你想加特别的Response Header。

- session Session
>

#####Render

***
rails的默认render，依照Rails惯例：app/views/{controller_name}/{action_name}。如果找不到样板档桉的话，会出现Template is missing的错误。

>直接回传结果
- render :text => "Hello" 直接回传字串内容，不使用任何样板。
- render :xml => @event.to_xml 回传XML格式
- render :json => @event.to_json 回传JSON格式(再加上:callback就会是JSONP)
- render :nothing => true 空空如也
>指定Template
- \:template 指定Template
- \:action 指定使用该Action的Template (只是使用它的Template，而不会执行该Action)
- \:file 指定Template的档名全名
> 其他参数
- \:status 设定HTTP status，默认是200，也就是正常。其他常用代码包括401权限不足、404找不到页面、500服务器错误等。
- \:layout 可以指定这个Action的Layout，设成false即关掉Layout
>在特定情况你想把render的结果存成一个字串，例如拿到局部样板Partials成为一个字串，这时候可以改使用render_to_string :partial => "foobar"

#####Redirect

***

如果Action不要render任何结果，而是要使用者转向到别页，可以使用redirect_to

	redirect_to :action => "show", :id => @event
	redirect_to :back 回到上一页。
注意，一个Action中只能有一个render或一个redirect_to。不然你会得到一个DoubleRenderError例外错误。

#####Sending data

回传二进制Binary资料，有两个方法可以使用：
>send_data(data, options={}) 回传二进制码流，接受以下参数：

>- data 是二进制码流
>- \:filename 存储的文件名
>- \:type 预设是application/octet-stream
>- \:disposition inline或attachment
>- \:status 预设是200
>send_file(file_location, options={}) 回传一个文件，接受以下参数：

>- file_location 文件路径和文件名
>- \:filename 存储的文件名
>- \:type 预设是application/octet-stream
>- \:disposition inline或attachment
>- \:status 预设是200
>- \:buffer_size stream的缓存空间，预设4096 bytes
>- \:stream 默认是false，会先将整个文件读入内存，如果文件非常大可能造成问题。

Rails支持-Sendfile

Header可以将传输任务委派给网页服务器处理，降低Rails服务器的负担。

#####respond_to
***
respond_to可以用来回应不同的资料格式。Rails内建支援格式包括有:html, :text, :js, :css, :ics, :csv, :xml, :rss, :atom, :yaml, :json等。如果需要扩充，可以编辑config/initializers/mime_types.rb这个档桉。

>如果你想要设定一个else的情况，你可以用:any：

	respond_to do |format|
	  format.html
	  format.xml { render :xml => @event.to_xml }
	  format.any { render :text => "WTF" }
	end
###Sessions
***
HTTP是一种无状态的通讯协定，为了能够让浏览器能够在跨request之间记住资讯，Rails提供了Session功能。

>要操作Session，直接操作session这个Hash变量即可。例如：
	session[:cart] = Cart.new
>只要是可以被序列化的对象，都可以放进session之中。

####Session存储

>Rails默认Cookies session storage来储存Session资料，它是将Session资料透过config/initializers/secret_token编码后放到浏览器的Cookie之中，最大的好处是对伺服器的效能负担很低，缺点是大小最多4Kb，以及资料还是可以透过反编码后看出来，只是无法进行修改。因此安全性较低，不适合存放机密资料。
>
>Rails支持的其他方式，你可以修改config/initializers/session_store.rb：
	:active_record_store 使用数据库来储存。
	:mem_cache_store 使用Memcached快取系统来储存

> > 採用:active_record_store的话，必须产生sessions资料表：
	$ rake db:sessions:create
	$ rake db:migrate
###Cookies
	# Sets a simple session cookie.
	cookies[:user_name] = "david"
	
	# Sets a cookie that expires in 1 hour.
	cookies[:login] = { :value => "XJ-122", :expires => 1.hour.from_now }
	
	# Example for deleting:
	cookies.delete :user_name
	
	cookies[:key] = {
	  :value => 'a yummy cookie',
	  :expires => 1.year.from_now,
	  :domain => 'domain.com'
	}
	
	cookies.delete(:key, :domain => 'domain.com')
因为资料是存放在使用者浏览器，所以如果需要保护不能让使用者乱改，Rails也提供了Signed方法：

	cookies.signed[:user_preference] = @current_user.preferences
另外，如果是尽可能永远留在使用者浏览器的资料，可以使用Permanent方法：

	cookies.permanent[:remember_me] = [current_user.id, current_user.salt]
两者也可以加在一起用：

	cookies.permanent.signed[:remember_me] = [current_user.id, current_user.salt]

###Flash讯息

用处在于redirect时，能够从这一个request传递文字讯息到下一个request

>flash是一个Hash，其中的键你可以自定，常用:notice、:warning或:error等

	def create
	  @event = Event.create(params[:event])
	  flash[:notice] = "成功建立"
	  redirect_to :action => :show
	end
在下一个Action中，我们就可以在Template中读取到这个讯息，通常我们会放在Layout中：

	<p><%= flash[:notice] %></p>
使用过一次之后，Rails就会自动清除flash。

另外，有时候你等不及到下一个Action，就想让Template在同一个Action中读取到flash值，这时候你可以写成：

	flash.now[:notice] = "foobar"
###Filters

可将Controller中重複的程式抽出来，有三种方法可以定义在进入Action之前、之中或之后执行特定方法，分别是before_filter、after_filter和around_filter，其中before_filter最为常用。这三个方法可以接受Code block、一个Symbol方法名称或是一个物件(Rails会呼叫此物件的filter方法)。

> before_filter
>
>before_filter最常用于准备跨Action共用的资料，或是使用者权限验证等等：
	class EventsControler < ApplicationController
	  before_filter :find_event, :only => :show
	  def show
	  end
	  
	  protected
	  
	  def find_event
		@event = Event.find(params[:id])
		end
		end
>每一个都可以搭配:only或:except参数。
>
>around_filter
	# app/controllers/benchmark_filter.rb
	class BenchmarkFilter
	  def self.filter(controller)
	    timer = Time.now
	    Rails.logger.debug "---#{controller.controller_name} #{controller.action_name}"
	    yield # 这裡让出来执行Action动作
	    elapsed_time = Time.now - timer
	    Rails.logger.debug "---#{controller.controller_name} #{controller.action_name} finished in %0.2f" % elapsed_time
	  end
	end

	# app/controller/events_controller.rb
	class EventsControler < ApplicationController
	  around_filter BenchmarkFilter
	end
####Filter的顺序

>当有多个Filter时，Rails是由上往下依序执行的。如果需要加到第一个执行，可以使用prepend_before_filter方法，同理也有prepend_after_filter和prepend_around_filter。
>
>如果需要取消从父类别继承过来的Filter，可以使用skip_before_filter :filter_method_name方法，同理也有skip_after_filter和skip_around_filter。

###rescue_from
>在Controller中处理异常
	class ApplicationController < ActionController::Base
	  rescue_from ActiveRecord::RecordNotFound, :with => :show_not_found
	  rescue_from ActiveRecord::RecordInvalid, :with => :show_error
	  
	protected
	  def show_not_found
        # render something
      end
	  def show_error
        # render something
      end
	end
###HTTP 基本认证
>Rails内建支援HTTP Basic Authenticate
	class PostsController < ApplicationController
      before_filter :authenticate
    protected
	  def authenticate
        authenticate_or_request_with_http_basic do |username, password|
		  username == "foo" && password == "bar"
        end
      end
	end
或是这样写：

	class PostsController < ApplicationController
      http_basic_authenticate_with :name => "foo", :password => "bar"
	end