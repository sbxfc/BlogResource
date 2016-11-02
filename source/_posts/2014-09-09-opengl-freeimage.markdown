---
layout: post
title: "在 macOS 下使用 FreeImage 库"
date: 2014-09-09 11:29:05 +0800
comments: true
categories: 
---


FreeImage是一个开源图片开发库,最后一个版本3.1.6。

#下载&编译

3.1.6版本我在Mac下没有编译通过,我用的是3.1.54版本。

3.1.54下载地址 : <http://sourceforge.net/projects/freeimage/>

编译:

	$ cd /Users/Downloads/FreeImage
	$ make

不出意外,编译会出错,因为Makefile.osx里的配置已经过期。
下载新的 Makefile.osx并替换:<br>
<https://github.com/sbxfc/FreeImage-Makefile.osx>

然后,在 Source/OpenEXR/IlmImf/ImfAutoArray.h 里添加:

	#include <cstring>

问题看这里↓:

<http://stackoverflow.com/questions/19080303/how-to-compile-freeimage-on-mac-os-x-10-8>

#运行

在使用FreeImage库时，需要设置一下Hearder Search paths和 Lib Search patchs,并在Build Phases里将libfreeimage.a添加进来。

#参见

- [3.1.6 下载地址](http://freeimage.sourceforge.net/download.html)