---
layout: post
title: sinatra_recipes
description: "sinatra, recipes"
modified: 2013-09-25
tags: [sinatra]
image:
  feature: texture-feature-04.jpg
comments: true
share: true
---

参考自[Sinatra Recipes](http://recipes.sinatrarb.com/)

github

{% highlight sh %}
git clone git://github.com/sinatra/sinatra-recipes.git
{% endhighlight %}

rackup, http://localhost:9292/ 查看

### Asset管理

CSS, JS, Image的管理

1. 编译(使用say, Sass, Coffeescript编写) => css或js脚本
2. 连接 多个文件连接到一个单一文件
3. 压缩 删除多余空格, 添加文件的大小, 更换函数内长变量名, 
4. 缓存清理 浏览器缓存机制确保文件不会每次都从服务器端下载, 缓存清理机制提供一个唯一的版本号码让浏览器下载最新版本asset 

#### sinatra-assetpack

### 数据库

不使用ORM直接使用操作数据库

    gem install mongo
    gem install bson_ext

{% highlight ruby %}
require 'rubygems'
require 'sinatra'
require 'mongo'
require 'json/ext' # required for .to_json

include Mongo

configure do
  conn = MongoClient.new("localhost", 27017)
  set :mongo_connection, conn
  set :mongo_db, conn.db('test')
end  
{% endhighlight %}

使用 settings.mongo_db和辅助方法操作mongoDB

### 部署

#### nginx + thin

设置 thin的 config.yml

{% highlight yaml %}
 environment: production
 chdir: /path/to/my/app
 address: 127.0.0.1
 user: root
 group: root
 port: 4567
 pid: /path/to/my/app/thin.pid
 rackup: /path/to/my/app/config.ru
 log: /path/to/my/app/thin.log
 max_conns: 1024
 timeout: 30
 max_persistent_conns: 512
 daemonize: true
 servers: 2
{% endhighlight %}

nginx 配置

{% highlight sh %}
upstream www_mydomain_com {
  server 127.0.0.1:5000;
  server 127.0.0.1:5001;
}

server {
  listen    www.mydomain.com:80
  server_name  www.mydomain.com live;
  access_log /path/to/logfile.log

  location / {
    proxy_pass http://www_mydomain_com;
  }

}
{% endhighlight %}

启动 thin和应用

{% highlight sh %}
thin -C config.yml -R config.ru start
{% endhighlight %}

### 嵌入式应用

嵌入sinatra到另一个框架

#### EventMachine

{% highlight ruby %}
require 'eventmachine'
require 'sinatra/base'
require 'thin'

def run(opts)

  # Start he reactor
  EM.run do

    # define some defaults for our app
    server  = opts[:server] || 'thin'
    host    = opts[:host]   || '0.0.0.0'
    port    = opts[:port]   || '8181'
    web_app = opts[:app]

    dispatch = Rack::Builder.app do
      map '/' do
        run web_app
      end
    end
    
    unless ['thin', 'hatetepe', 'goliath'].include? server
      raise "Need an EM webserver, but #{server} isn't"
    end
    
    Rack::Server.start({
      app:    dispatch,
      server: server,
      Host:   host,
      Port:   port
    })
  end
end

# Our simple hello-world app
class HelloApp < Sinatra::Base
  # threaded - False: Will take requests on the reactor thread
  #            True:  Will queue request for background thread
  configure do
    set :threaded, false
  end

  # Request runs on the reactor thread (with threaded set to false)
  get '/hello' do
    'Hello World'
  end

  # Request runs on the reactor thread (with threaded set to false)
  # and returns immediately. The deferred task does not delay the
  # response from the web-service.
  get '/delayed-hello' do
    EM.defer do
      sleep 5
    end
    'I\'m doing work in the background, but I am still free to take requests'
  end
end

# start the application
run app: HelloApp.new
{% endhighlight %}

启动

{% highlight sh %}
ruby em-sinatra-test.rb
{% endhighlight %}

使用ab测试

{% highlight sh %}
ab -c 10 -n 100 http://localhost:8181/delayed-hello
{% endhighlight %}

### 辅助方法

#### Partials

提供rails风格的partials

#### 使用sinatra-partial gem

### Model

ORM方法适配

#### mongoid

### 测试

#### Rspec

设置Rack::Test

{% highlight ruby %}
# spec/spec_helper.rb
require 'rack/test'

require File.expand_path '../../my-app.rb', __FILE__

module RSpecMixin
  include Rack::Test::Methods
  def app() described_class end
end

# For RSpec 2.x
RSpec.configure { |c| c.include RSpecMixin }
# If you use RSpec 1.x you should use this instead:
# Spec::Runner.configure { |c| c.include RSpecMixin }
{% endhighlight %}

spec文件中

{% highlight ruby %}
# spec/app_spec.rb
require File.expand_path '../spec_helper.rb', __FILE__

describe "My Sinatra Application" do
  it "should allow accessing the home page" do
    get '/'
    last_response.should be_ok
  end
end
{% endhighlight %}

#### Cucumber 和 Webrat 

示例

{% highlight ruby %}
Feature: View my page
  In order for visitors to feel welcome
  We must go out of our way
  With a kind greeting

  Scenario: My page
    Given I am viewing my page
    Then I should see "Welcome to my page!"
{% endhighlight %}

{% highlight ruby %}
Given /^I am viewing my page$/ do
  visit('/')
end

Then /^I should see "([^"]*)"$/ do |text|
  last_response.body.should match(/#{text}/m)
end
{% endhighlight %}

配置

{% highlight ruby %}
# features/support/env.rb

ENV['RACK_ENV'] = 'test'

require 'rubygems'
require 'rack/test'
require 'rspec/expectations'
require 'webrat'

require File.expand_path '../../../my-app.rb', __FILE__

Webrat.configure do |config|
  config.mode = :rack
end

class WebratMixinExample
  include Rack::Test::Methods
  include Webrat::Methods
  include Webrat::Matchers

  Webrat::Methods.delegate_to_session :response_code, :response_body

  def app
    Sinatra::Application
  end
end

World{WebratMixinExample.new}
{% endhighlight %}

### Views

### 中间件

#### Rack::Auth

#### Rack::CommonLogger

输出到stdout和文件

{% highlight ruby %}
require 'sinatra/base'

class SomeApp < Sinatra::Base
  configure do
    enable :logging
    file = File.new("#{settings.root}/log/#{settings.environment}.log", 'a+')
    file.sync = true
    use Rack::CommonLogger, file
  end

  get '/' do
    'Hello World'
  end

  run!
end
{% endhighlight %}

#### Memcached

sinatra部署到多台服务器上, 共享缓存, 使用dalli作为memcached客户端

{% highlight ruby %}
configure do
  use Rack::Session::Dalli,
    memcache_server: 'example.com:11211',
    namespace: 'other.namespace'
    cache: Dalli::Client.new
end
{% endhighlight %}
