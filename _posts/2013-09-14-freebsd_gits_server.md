---
layout: post
title: freebsd架设git服务器
description: "架设git服务器"
modified: 2013-09-14
tags: [freebsd]
image:
  feature: texture-feature-03.jpg
comments: true
share: true
---

安装git

### 创建目录

{% highlight sh %}
mkdir -p /usr/local/git
chown -R git_daemon:git_daemon /usr/local/git
chmod -R 0755 /usr/local/git
{% endhighlight %}

####  创建ssh目录

{% highlight sh %}
mkdir -p /usr/local/git/.ssh
touch /usr/local/git/.ssh/authorized_keys
chown -R git_daemon:git_daemon /usr/local/.ssh
chmod 0700 /usr/local/git/.ssh
chmod 0600 /usr/local/git/.ssh/authorized_keys
{% endhighlight %}

###启动git server

配置 /etc/rc.conf, 然后

{% highlight sh %}
/usr/local/etc/rc.d/git_daemon start
{% endhighlight %}

###建立repoitory

{% highlight sh %}
cd /usr/local/git
mkdir test.git
cd test
git --bare init
chown -R git_daemon:git_daemon ../test.git
{% endhighlight %}

###使用git

{% highlight sh %}
git clone user@server:/usr/local/git/test.git
{% endhighlight %}