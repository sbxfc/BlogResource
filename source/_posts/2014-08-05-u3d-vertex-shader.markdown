---
layout: post
title: "Unity - 顶点函数"
date: 2014-08-05 11:08:29 +0800
comments: true
categories: 
---

在声明一个表面着色器的时,我们可以同时使用vertex:指定一个顶点函数。
顶点函数可以使我们在着色器绘制顶点时有机会去修改顶点数据。<br>
顶点函数接收一个appdata_full结构的顶点数据,包括顶点做坐标、颜色和法线等信息。我们可以根据这些内容，结合其他一些有用的信息，创造出丰富的效果。

	#pragma surface surf Lambert vertex:vert

在每一帧的顶点绘制时,我们根据_Time的时间偏移和正弦函数重新计算了顶点的y坐标,使得顶点的高度有一个正弦变化。

顶点着色器代码:<br>
<https://github.com/rungameme/u3dVertexShader>