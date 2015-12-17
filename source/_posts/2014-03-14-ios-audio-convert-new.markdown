---
layout: post
title: "iOS - Mac下音频格式转换"
date: 2014-03-14 10:45:07 +0800
comments: true
categories: 
---
#查看音频信息

首先,cd到文件所在目录下,然后输入<font color='#bd260d'>**afinfo**</font>指令查看文件信息

	afinfo 李荣浩-老街.mp3
	
输出结果:
	
	File:           李荣浩-老街.mp3
	File type ID:   MPG3
	Num Tracks:     1
	----
	Data format:     2 ch,  44100 Hz, '.mp3' (0x00000000) 0 bits/channel, 0 bytes/packet, 1152 frames/packet, 0 bytes/frame
	                no channel layout.
	estimated duration: 318.824475 sec
	audio bytes: 12752979
	audio packets: 12205
	bit rate: 320000 bits per second
	packet size upper bound: 1052
	maximum packet size: 1045
	audio data file offset: 10939
	optimized
	audio 14059008 valid frames + 576 priming + 576 remainder = 14060160
	----

#格式转化

使用<font color='#bd260d'>**afconvert**</font>命令可将音频文件转化为指定格式。

格式如下:

	afconvert -d [out data format] -f [out  file format] [in file] [out file]
	
以转化IMA4压缩格式的CAF音频文件为例:（参见[iOS首选音频格式](http://rungame.me/blog/2014/02/14/iOS-audio/ "")）

	afconvert -d ima4 -f caff 李荣浩-老街.mp3 李荣浩-老街.caf
	
其中,`-d` 指定压缩格式,`-f` 指定文件格式

#批量转化

在包含音频文件的目录下:

	
	find . -name '*.mp3' -exec afconvert -f caff -d ima4 {} \;

