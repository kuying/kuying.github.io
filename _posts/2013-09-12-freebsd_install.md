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

常用 bash vim ruby ruby-gem mongo nigix git

mongodb 默认使用 /var/db, 配置文件在/usr/local/etc/mongodb.conf

修改 /etc/rc.conf

{% highlight sh %}
mongod_enable="YES"
{% endhighlight %}

修改 bash

{% highlight sh %}
chsh -s /usr/local/bin/bash
{% endhighlight %}

同步时间

{% highlight sh %}
ntpdate nist1.symmetricom.com
{% endhighlight %}

### 安装samba

{% highlight sh %}
/usr/ports/net/samba36
{% endhighlight %}

配置

{% highlight sh %}
   security = share
;   hosts allow = 192.168.1. 127.
   load printers = no
;  guest account = nobody 
;   interfaces = 192.168.1.100/24 192.168.10.1/24 
;   domain master = no 
;   preferred master = no
;   domain logons = no
;   wins support = no
;   wins proxy = no
[public]
   comment = Public
   browseable = yes
   path = /samba
   guest ok = yes
   read only = no
   public = yes
   printable = no
{% endhighlight %}
   
### 升级

{% highlight sh %}

portsnap update

portupgrade -ai
{% endhighlight %}

