---
layout: post
title: freebsd安装
description: "重新安装freebsd, 备忘"
modified: 2013-09-12
tags: [os]
image:
  feature: abstract-4.jpg
comments: true
share: true
---

光盘安装freebsd

### 配置网卡

{% highlight sh %}
/etc/rc.conf
hostname="freebsd"
ifconfig_re0=" inet 192.168.10.1 netmask 255.255.255.0"
# Wireless config
wlans_run0="wlan0"
ifconfig_wlan0="WPA DHCP"
{% endhighlight %}

#### 无线网卡

{% highlight sh %}
/etc/wpa_supplicant.conf 
network={
  ssid="TP-LINK_xx"
  psk="xxxxxxxxx"
}
{% endhighlight %}

重启

{% highlight sh %}
etc/rc.d/netif start
{% endhighlight %}

###配置sshd

{% highlight sh %}
/etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
{% endhighlight %}

重启

{% highlight sh %}
etc/rc.d/sshd start
{% endhighlight %}

###升级freebsd

{% highlight sh %}
freebsd-update fetch
freebsd-update install
{% endhighlight %}

###安装port

{% highlight sh %}
portsnap fetch extract update 
{% endhighlight %}

使用axel加速port

修改

{% highlight sh %}
#vi /etc/make.conf

  FETCH_CMD=axel -a

  DISABLE_SIZE=yes     

  MASTER_SIDE_OVERRIDE=

  ftp://ftp2.tsinghua.edu.cn/mirror/FreeBSD/ports/distfiles /

  ftp://freebsd.csie.nctu.edu.tw/pub/FreeBSD/ports/distfiles /

  ftp://ftp.hk.freebsd.org/pub/FreeBSD/ports/distfiles /

  ftp://ftp.freebsdchina.org/pub/FreeBSD/ports/distfiles / 
{% endhighlight %}

修改vi /usr/local/etc/axelrc 

{% highlight sh %}
num_connections = 10 
{% endhighlight %}

###安装

查找

{% highlight sh %}
echo /usr/ports/*/*/port_name
{% endhighlight %}

安装

{% highlight sh %}
make install clean
{% endhighlight %}

常用 bash vim ruby mongo nigix

修改 bash

{% highlight sh %}
chsh -s /usr/local/bin/bash
{% endhighlight %}

同步时间

{% highlight sh %}
ntpdate nist1.symmetricom.com
{% endhighlight %}

