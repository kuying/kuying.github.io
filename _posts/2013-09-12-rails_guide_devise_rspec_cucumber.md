---
layout: post
title: Rails练习(Devise,RSpec,Cucumber)
description: "rails demo, use devise with rspec and cucumber."
modified: 2013-09-12
tags: [rails]
image:
  feature: texture-feature-02.jpg
comments: true
share: true
---


###Rspec单元测试

***

首选的rails测试框架

####gem

{% highlight ruby %}
gem 'rspec-rails', :group => [:development, :test]

group :test do
  gem 'database_cleaner'
  gem 'factory_girl_rails'
  gem 'mongoid-rspec'
end
{% endhighlight %}

####生成Rspec

{% highlight sh %}
rails g rspec:install
{% endhighlight %}

生成.rspec 和spec/spec_helper.rb文件

注释spec_helper文件中用于ActiveRecord的部分, 以及

{% highlight ruby %}
# config.fixture_path = "#{::Rails.root}/spec/fixtures"
# config.use_transactional_fixtures = true
{% endhighlight %}

删除test文件夹, Rspec不需要

默认情况下 rails生成器(rails generate controller/scaffold )为view和helper生成spec文件, 如果不测试views和helpers, 可以关闭之

{% highlight ruby %}
class Application < Rails::Application
  config.generators do |g|
    g.view_specs false
    g.helper_specs false
  end
{% endhighlight %}

####数据清理database_cleaner

每次测试之前需要清理数据库, 配置spec_helper

{% highlight ruby %}
RSpec.configure do |config|
  # Other things

  # Clean up the database
  require 'database_cleaner'
  config.before(:suite) do
    DatabaseCleaner.strategy = :truncation
    DatabaseCleaner.orm = "mongoid"
  end

  config.before(:each) do
    DatabaseCleaner.clean
  end
end
{% endhighlight %}

####RSpec适配器 mongoid-rspec

方便测试mongoid的校验器, 新增 spec/support/mongoid.rb

{% highlight ruby %}
RSpec.configure do |config|
  config.include Mongoid::Matchers
end
{% endhighlight %}

建议使用Cucumber, 减少对ORM的依赖

####FactoryGirl

为测试创建默认model对象, 如果一个controller的action在show之前要先查找一个User对象, Factory Girl将为之创建一个user

####Devise test helper

使用Devise后, 一般在controller中会使用 before_filter :authenticate_user! 要求用户登入, 这个将导致测试失败

配置 spec/support/devise.rb

{% highlight ruby %}
RSpec.configure do |config|
  config.include Devise::TestHelpers, :type => :controller
end
{% endhighlight %}

### 运行

运行测试

{% highlight sh %}
rake spec
{% endhighlight %}

### Cucumber BDD

####gem

{% highlight ruby %}
group :test do
  gem 'cucumber-rails'
  gem 'capybara'
  gem 'database_cleaner'
end
{% endhighlight %}

生成Cucumber的必要文件 config/cucumber.yml script/cucumber features/support/env.rb lib/tasks/cucumber.rake 

{% highlight sh %}
rails generate cucumber:install --capybara --rspec --skip-database
{% endhighlight %}

--capybara 指定Capybara替代默认的Webrat来接收测试, 

--rspec只是使用RSpec适配器

--skip-database 在-o生成application时, 不检查database.yml

####Database Cleaner

修改 feature/support/env.rb

{% highlight ruby %}
begin
  DatabaseCleaner.orm = 'mongoid'
  DatabaseCleaner.strategy = :truncation
rescue NameError
  raise "You need to add database_cleaner to your Gemfile (in the :test group) if you wish to use it."
end
{% endhighlight %}

####运行

{% highlight sh %}
rake cucumber
{% endhighlight %}

####Cucmber用例

- xxx.feature 文件: 通过一些具有代表性的例子来描述一个用户需求， 
- KeyExamples: 关键用例，特性之间都可以通过自己的关键用例加以区分，每个关键用例都有明确的输入和输出。 
- Scenario: 测试场景，一个用户特性的一个关键用例就称之为一个测试场景。 
- Step: 测试步骤，一个测试场景涉及到多个步骤操作， 
- Step_Definitions: 步骤定义，定义测试用例中步骤的执行顺序。 
- Gherkin: 用来定义特性能够文件的结构和关键字含义的语言

###配置ActionMailer

{% highlight sh %}
rails g figaro:install
{% endhighlight %}

生成 config/application.yml, 并设置.gitignore

在 config/application.yml中设置mail的用户名/密码

在config/environments/production.rb 和 config/environments/development.rb中分别设置

{% highlight ruby %}
  # config.action_mailer.raise_delivery_errors = false # for production
  config.action_mailer.raise_delivery_errors = false # for development

  config.action_mailer.smtp_settings = {
    address: "smtp.gmail.com",
    port: 587,
    domain: ENV["DOMAIN_NAME"],
    authentication: "plain",
    enable_starttls_auto: true,
    user_name: ENV["GMAIL_USERNAME"],
    password: ENV["GMAIL_PASSWORD"]
  }
{% endhighlight %}

### 配置devise鉴权

{% highlight sh %}
rails g devise:install
{% endhighlight %}

生成 config/initializers/devise.rb 和 config/locales/devise.en.yml

devise自动识别mongoid

在config/environments/production.rb 和 config/environments/development.rb中设置default_url_options

{% highlight ruby %}
config.action_mailer.default_url_options = { :host => 'localhost:3000' } # set to host.com at production
{% endhighlight %}

####自动生成model

{% highlight sh %} 
rails g devise User
{% endhighlight %}

创建 app/model/user.rb 和 spec/model/suser_spec.rb spec/factories/users.rb, 修改路由 devise_for :users

####sign out问题

devise默认使用DELETE做sign out请求, Cucumber使用GET测试, 需要修改Devise来适配

修改 config/initializer/devise.rb

{% highlight ruby %}
# The default HTTP method used to sign out a resource. Default is :delete.
config.sign_out_via = Rails.env.test? ? :get : :delete
{% endhighlight %}

password不记日志

修改app/application.rb 

{% highlight ruby %}
  config.filter_parameters += [:password, :password_confirmation]
{% endhighlight %}

### 自定义修改部分

增加字段

执行

{% highlight sh %}
rails g devise:views
{% endhighlight %}

生成view在修改之


