---
layout: post
title: "BIND9服务器"
date: 2016-11-17 16:03:05 +0800
comments: true
categories: 
---

如果想要为你的顶级域名(xxx.com)创建域名服务器,根据DNS协议你需要向上一级域名服务器(.com)申请。就像我们的域名申请一样,这些操作都是可以通过注册商来完成的,比如阿里,比如狗爹(Godaddy)。阿里申请一个域名服务器要花10RMB,一般注册商要求配置俩域名服务器。注册DNS服务器时,阿里没有检测这台主机的IP是否有DNS服务器,所以填写的时候可以随便填写。我的域名下,ns.ok2.site这个DNS是有效的,ns2.ok2.site这个DNS是无效的,但并不影响解析。

关于DNS的安装和配置,网络上都是有的。只不过配置起来挺繁琐,但实际上没有什么难度。可以参见 《DNS&BIND 4th》,这本书讲的很详细，虽然是挺老的书,但是对DNS来说,变化不大。

#下载

下载最后一版本的BIND9:

	$wget -c ftp.isc.org/isc/bind9/9.9.9/bind-9.9.9.tar.gz
	
#安装
	
	$tar zxvf bind-9.9.9.tar.gz -C /usr/local/src/ 
	$cd /usr/local/src/bind-9.9.9/ 
	$sudo ./configure --prefix=/usr/local/bind-9.9.9 --enable-threads --disable-openssl-version-check --sysconfdir=/etc --with-libtool
	$sudo make 
	$sudo make install
	
.configure参数的说明:

参数 | 说明 
------------ | -------------  
--prefix=/usr/local/bind |指定bind9的安装目录,默认值是/usr/local
--enable-threads |开启多线程的支持；如果你的系统有多个CPU，那么可以使用这个选项
--disable-openssl-version-check | 关闭openssl的版本检查
--with-openssl=/usr/local/openssl | 指定openssl的安装路径
--sysconfdir=/etc/bind |设置named.conf配置文件放置的目录，默认是--prefix选项指定的目录下的/etc下
--localstatdir=/var |设置 run/named.pid 放置的目录，默认是--prefix选项指定的目录下的/var下
--with-libtool | 将BIND的库文件编译为动态共享库文件，这个选项默认是未选择的。如果不选这个选项，那么编译后的named命令会比较大，lib目录中的库文件都是.a后缀的。如果选上这个选项，那么编译后的named命令会很小，lib目录中的库文件则是.so后缀

问题1:

<font color='#bd260d'>*checking for OpenSSL library... configure: error: OpenSSL was not found in any of /usr /usr/local /usr/local/ssl /usr/pkg /usr/sfw; use --with-openssl=/path
If you don't want OpenSSL, use --without-openssl*</font>

解决:
	
	$ yum install openssl-devel
	
#配置BIND

1,BIND9安装成功之后,首先创建BIND9的配置文件,直接保存退出即可:

	$vi /etc/named.conf 

2,创建区数据文件目录 

	$mkdir /var/named	
	
#建立区数据文件

区数据文件的大部分条目称为DNS资源记录(resouce record)。我们把区数据文件的根目录设置在了 <font color='#bd260d'>*/var/named/*</font> 建立 ok2.site 区的数据文件 <font color='#bd260d'>*db.ok2.site*</font>


	$TTL  3d
	@                   IN      SOA     ns.ok2.site. admin.ok2.site. (
	                                                          3     ;序列号
	                                                          3h    ;3小时后刷新
	                                                          1h    ;1小时后重试
	                                                          1w    ;1周后期满
	                                                          1h)   ;否定缓存TTL为1小时
	@                   IN      NS      ns.ok2.site.
	@                   IN      NS      ns1.ok2.site.
	@                   IN      MX 10   mail.ok2.site.    ;本域中邮件转递的主要邮件主机，后面的数字越小，优>先级越高。
	ns                  IN      A       35.164.14.188
	ns1                 IN      A       123.57.38.225
	www                 IN      A       35.164.14.188
	mail                IN      A       35.164.14.188
	ftp                 IN      CNAME   www               ;指向主机www的别名


TTL 值得全称是 Time To Live,即存活时间。这里是指允许其他服务器将区数据在缓存中存放的时间。如果数据不是经常变动可以将值设置为几天。

SOA 记录的全称是 Start Of Authority,该值指定该区的权威服务器。

#BIND配置文件

BIND服务器启动时会首先加载配置文件,然后根据配置文件里的信息去读取区数据文件。配置文件的默认位置在 <font color='#bd260d'>*/etc/named.conf*</font> 

	options
	{
        directory "/var/named"; #区数据文件所在目录
        pid-file "named.pid";
        forwarders {8.8.8.8;};
        dump-file "cache_dump.db";
        statistics-file "named_stats.txt";
        allow-query    { any; };
	};
	
	logging
	{
        channel default_debug {
        file "named.run";
        severity dynamic;};
	};
	
	zone "ok2.site" IN 
	{
		type master;
		file "db.ok2.site";
		allow-update {none;};
	};
	
	zone "localhost" IN
	{
        type master;
        file "db.localhost";
        allow-update {none;};
	};
	
	zone "14.164.35.in-addr.arpa" IN {
        type master;
        file "db.35.164.14";
        allow-update { none; };
	};
	
	zone "0.0.127.in-addr.arpa" IN
	{
        type master;
        file "db.127.0.0";
        allow-update {none;};
	};
	
	zone "." IN
	{
        type hint;
        file "db.root";
	};
	
#启动

	# /usr/local/bind-9.9.9/sbin/named -g

#参见

- [DNS&BIND 4th.pdf]()