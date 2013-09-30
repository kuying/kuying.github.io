---
layout: post
title: sinatra使用rspec测试
description: "sinatra测试, rspec"
modified: 2013-09-25
tags: [sinatra]
image:
  feature: texture-feature-05.jpg
comments: true
share: true
---

参考自[Sinatra Test](http://www.sinatrarb.com/testing.html)

### Rack::Test

#### Rack::Test::Methods

Rack::Test::Methods 提供了一些常用的辅助方法, mixin方法引用之

	request
	get
	post
	put
	patch
	delete
	options
	head
	follow_redirect!
	header
	env
	set_cookie
	clear_cookies
	authorize
	basic_authorize
	digest_authorize
	last_response
	last_request
  
#### Mock方法

	get, put, post, delete, head模拟各种类型的request

{% highlight ruby %}
get '/path', params={}, rack_env={}
{% endhighlight %}

- /path 请求URL.
- params query/post参数的hash, 表示请求body的字符串, 或为空
- rack_env Rack环境参数的hash. 可以用来设置请求头和其他请求相关信息, 如session

####  检查响应

app 处理mock请求
last_request 使用Rack::MockRequest生成一个请求
last_response 使用Rack::MockResponse实例化应用产生的响应

#### 设置

使用 sinatra/base 时

{% highlight ruby %}
def app
   MySinatraApp
end
{% endhighlight %}

### Rspec

使用 sinatra/base 时

{% highlight ruby %}
ENV['RACK_ENV'] = 'test'

require 'hello_world'  # <-- your sinatra app
require 'rspec'
require 'rack/test'

describe 'The HelloWorld App' do
  include Rack::Test::Methods

  def app
    Sinatra::Application
  end

  it "says hello" do
    get '/'
    expect(last_response).to be_ok
    expect(last_response.body).to eq('Hello World')
  end
end
{% endhighlight %}

配置

{% highlight ruby %}
require 'rspec'
require 'rack/test'

RSpec.configure do |conf|
  conf.include Rack::Test::Methods
end
{% endhighlight %}