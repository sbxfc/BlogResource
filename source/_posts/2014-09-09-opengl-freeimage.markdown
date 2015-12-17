---
layout: post
title: "使用FreeImage库(Mac OS)"
date: 2014-09-09 11:29:05 +0800
comments: true
categories: 
---


FreeImage是一个开源图片开发库,最后一个版本3.1.6。

下载地址:

<http://freeimage.sourceforge.net/download.html>

编译FreeImage库:

	$ cd /Users/Downloads/FreeImage
	$ make

3.1.6版本我在Mac下没有编译通过,我用的是3.1.54版本并做了两次修改:

3.1.54下载地址:<br>
<http://sourceforge.net/projects/freeimage/>

一,正常编译会出错,因为Makefile.osx里的配置已经过期。
下载并替换掉包里的Makefile.osx文件:<br>
<https://github.com/opengl-records/FreeImage-Makefile.osx>

二,在 Source/OpenEXR/IlmImf/ImfAutoArray.h 里,添加:

	#include <cstring>

参见:

<http://stackoverflow.com/questions/19080303/how-to-compile-freeimage-on-mac-os-x-10-8>

使用FreeImage库时，需要设置一下Hearder search paths和 Lib search patchs,并在Build Phases里将libfreeimage.a添加进来。
