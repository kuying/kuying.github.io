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

~~~ sh
/etc/rc.conf
hostname="freebsd"
ifconfig_re0=" inet 192.168.10.1 netmask 255.255.255.0"
# Wireless config
wlans_run0="wlan0"
ifconfig_wlan0="WPA DHCP"
~~~~

#### 无线网卡

~~~ sh
/etc/wpa_supplicant.conf 
network={
  ssid="TP-LINK_xx"
  psk="xxxxxxxxx"
}
~~~

重启

~~~ sh
etc/rc.d/netif start
~~~

###配置sshd

~~~ sh
/etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
~~~

重启

~~~ sh
etc/rc.d/sshd start
~~~

###升级freebsd

~~~ sh
freebsd-update fetch
freebsd-update install
~~~

###安装port

~~~ sh
portsnap fetch extract update 
~~~

使用axel加速port

~~~ sh
make install clean
~~~

修改

~~~ sh
#vi /etc/make.conf

  FETCH_CMD=axel -a

  DISABLE_SIZE=yes     

  MASTER_SIDE_OVERRIDE=

  ftp://ftp2.tsinghua.edu.cn/mirror/FreeBSD/ports/distfiles /

  ftp://freebsd.csie.nctu.edu.tw/pub/FreeBSD/ports/distfiles /

  ftp://ftp.hk.freebsd.org/pub/FreeBSD/ports/distfiles /

  ftp://ftp.freebsdchina.org/pub/FreeBSD/ports/distfiles / 
~~~

修改vi /usr/local/etc/axelrc 

~~~ sh
num_connections = 10 
~~~


安装 bash vim ruby mongo nigix
