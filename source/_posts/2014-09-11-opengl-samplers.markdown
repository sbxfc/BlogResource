---
layout: post
title: "OpenGL - 采样器"
date: 2014-09-11 15:53:26 +0800
comments: true
categories: 
---

采样器是OpenGL 3.3中加入的特性。<br>
在此之前,当你使用同一张纹理但过滤方式不同时,你需要创建两个不同的纹理对象,然后分别设置不同的过滤。
使用采样器之后,你可以将过滤等属性设置绑定到一个特定的采样器上，然后在使用纹理时,绑定与相应的采样器绑定即可。


一,定义并设置采样器:

	glGenSamplers(1,&g_Sampler1);
    glSamplerParameteri(g_Sampler1, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glSamplerParameteri(g_Sampler1, GL_TEXTURE_MIN_FILTER, GL_LINEAR);

二,使用时与纹理绑定:

	glActiveTexture(GL_TEXTURE0);
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, g_texture1);
    glUniform1i(textureLocation1, 0);
    glBindSampler(0, g_Sampler1);

*PS.采样器只设定过滤，寻址模式等属性。*



#完整代码:

- [OpenGL-纹理映射](http://rungame.me/blog/2014/09/11/opengl-texture-mapping/)

- <https://github.com/sbxfc/OpenGLSamplers>