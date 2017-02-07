---
layout: post
title: "在亚马逊EC2 RHEL主机上安装nginx"
date: 2017-01-22 18:13:21 +0800
comments: true
categories: 
---

首先登陆亚马逊的EC2控制台,把安全组里的入站端口HTTP服务的默认端口80打开。

下载并安装EPEL:

	$wget http://download.fedoraproject.org/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm
	$rpm -ivh epel-release-5-4.noarch.rpm

安装nginx,看一眼安装提示,没问题就确认下载:

	$yum install nginx
	
启动nginx

	$/etc/init.d/nginx start	

然后就可以直接输入地址在浏览器里测试了,现在的首页是nginx的欢迎页面。

#配置虚拟主机

建立静态资源目录:

	$mkdir -p /var/www/ok2.site/public
	
在该public目录下建立首页index.html文件:

	<html>
	<body>
	<h1> HOST :)</h1>
	</body>
	</html>

修改www目录的访问权限:
	
	$chmod 755 /var/www

设置虚拟主机的配置文件:
	
	$vi /etc/nginx/conf.d/ok2.site.conf	
	
	#
	# A virtual host using mix of IP-, name-, and port-based configuration
	#
	
	server {
	    listen        80;
	#    listen       somename:8080;
	    server_name   ok2.site;
	
	    location / {
	        root   /var/www/ok2.site/public;
	        index  index.html index.htm;
	    }
	}

重启服务:

	$service nginx restart

当然,你可以根据不同的域名配置多台虚拟主机 :)
	
![](/images/2017/01/tmp01276c3d.png)
	

#参见:

- <http://comtechies.com/how-to-install-and-configure-nginx-on-amazon-ec2-rhel-and-ubuntu-instances.html>