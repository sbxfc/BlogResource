---
layout: post
title: "OpenGL - 纹理映射 "
date: 2014-09-11 09:58:11 +0800
comments: true
categories: 
---

#纹理映射

纹理映射就是将纹理上的像素映射到屏幕像素的过程。

和处理顶点数据不同,纹理对象在顶点、像素着色器的任意次调用中,都可以访问整个纹理对象。而顶点缓冲数据只是一次一个元素地提供给顶点着色器，并且顶点着色器没有任何方式访问其他的元素。为此，OpenGL提供了专门用于处理纹理的对象结构,用于存储纹理数据。像GLTEXTURE1D、GLTEXTURE2D、GLTEXTURE3D,我们经常使用的2D纹理用到的是GLTEXTURE2D对象。<br>

#纹理坐标

对于一张512\*512大小的2D 纹理,它是一个width*height个像素值的二维数组。为了获取某个具体的像素值，我们会使用他们的横、纵向的坐标来获取该单元。区别于空间坐标系，OpenGL将纹理空间的坐标系记为s、t、r。r是代表depth的值,因为除了传统的2D纹理，OpenGL还支持3D纹理，即GLTEXTURE3D。<br>

在纹理空间中，纹理数组里的元素均匀分布在纹理空间中。对于2D纹理，纹理空间一个从[0,0]到[1,1]的正方形坐标区间。对于1D纹理、3D纹理，纹理空间分别是从0到1的线性区间和从[0，0，0]到[1，1，1]的划分。其中，边界[0，0]映射2D纹理的第一个元素，随后沿着s坐标轴和t坐标轴向右、向上分布。

在一张512*512的纹理中，如果一个顶点的纹理坐标是[0.2,0.1],那么它在该纹理采样之后得到的纹理单元就是[512x0.2,512x0.1]即[102.4,51.2]。在OpenGL里提供了多种纹理过滤方式。最容易理解的是GL_NEAREST,即直接取整的方式，这时顶点获取的纹理单元就是[102，51]。但这种方式并不理想，会产生锯齿现象，这些会在纹理过滤中讲到。

#创建纹理

首先生成纹理名称,和其他OpenGL对象一样，使用glGen{}的形式返回该纹理的唯一名字mTextureId,然后通过glActiveTexture函数激活相应的纹理单元,GPU上只有有限数量的纹理单元可用于渲染:

	glGenTextures(1, &mTextureId);
	glActiveTexture(GL_TEXTURE0);
	
和其他OpenGL对象一样,创建一个纹理对象之后,为了操作该纹理对象,我们必须绑定该纹理对象:

	glBindTexture(GL_TEXTURE_2D, mTextureId);
	
当我们绑定当前纹理之后,我们就可以对当前的纹理对象进行操作。我们通过glTexImage2D函数,向显卡内存上传指定纹理的纹理数据:

	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, imgBuf);

使用glTexImage2D(1D、3D)函数指定纹理的格式和数据源，并为纹理对象分配内存。不像glBufferData,glTexImage2D要求对分配内存的所有的格式信息预先提出,并且使用该方法纹理贴图的每一维的大小必须是2的幂，包括边框

	void glTexImage2D(
			GLenum target,  
			GLint level,  
			GLint internalformat,  
			GLsizei width,  
			GLsizei height,  
			GLint border,  
			GLenum format,  
			GLenum type,  
			const GLvoid * data
		);

- target:指定纹理单元的类型。可选择的值GL_TEXTURE_2D,GL_TEXTURE_CUBE_MAP_POSITIVE_X,GL_TEXTURE_CUBE_MAP_NEGATIVE_X,GL_TEXTURE_CUBE_MAP_POSITIVE_Y,GL_TEXTURE_CUBE_MAP_NEGATIVE_Y,GL_TEXTURE_CUBE_MAP_POSITIVE_Z,GL_TEXTURE_CUBE_MAP_NEGATIVE_Z,这里我们使用的是2D纹理。

- level:指定层次细节(LOD)。一个纹理对象可能包含多个分辨率的相同图像，这些图像称作mipmap层，每个mipmap层数都有一个LOD索引，范围从0到最高分辨率。当从更低分辨率取样时可以依次从更小的"mipmaps"层次中取样。现在只有一个mipmap层，所以该值为0。

- internalformat:指定纹理的内部格式,internalformat参数告诉GPU每个纹理元素使用的颜色组分，以及以什么样的精度存储。可选择的值GL_ALPHA,GL_LUMINANCE,GL_LUMINANCE_ALPHA,GL_RGB,GL_RGBA.

- width/height:纹理的高度和宽度。宽度和高度参数指定纹理元素在s和t坐标轴上的数目。

- border:边界的宽度。(border参数在OpenGL ES上是废弃的并且总是应该设置为0)。

- format:源文件格式,源文件格式参数声明了我们的像素的组成顺序。(而且在OpenGL ES里要求和internalformat参数值相同。)

- type:指定数据类型。以下是OpenGL ES要求的类型,GL_UNSIGNED_BYTE,GL_UNSIGNED_SHORT_5_6_5,GL_UNSIGNED_SHORT_4_4_4_4,GL_UNSIGNED_SHORT_5_5_5_1

- data:纹理数据在内存中的指针

**uniform变量**
	
	glUniform1i(textureLocation,0);

通过uniform变量来设置纹理单元索引。<br>
uniform变量在shader里不能被修改，只能通过外部修改其值。<br>
使用glUniform*函数给uniform变量赋值。<br>
函数格式:<br>

	glUniform{dim}{type}

- dim:表示类型的大小(比如,int或float的是1，vec2是2，vec3是3，等等)
- type对应相应的类型：要么是i表示integer，要么是f表示float。

#纹理过滤
	
在纹理映射过程中，纹理元素与屏幕上的像素几乎不是一一对应的。一般情况下，在把纹理映射到几何图形的表面上时，我们需要对其进行拉伸或收缩。对一个纹理贴图的拉伸或收缩来计算颜色片段的过程称为纹理过滤。

	void glTexParameteri(	
				GLenum 	target,
				GLenum pname,
				GLint param
			);


- target :操作的纹理类型。GL_TEXTURE_2D(2D纹理),GL_TEXTURE_CUBE_MAP

- pname : 参数类型GL_TEXTURE_MIN_FILTER,GL_TEXTURE_MAG_FILTER,GL_TEXTURE_WRAP_S,GL_TEXTURE_WRAP_T
	
- param : 参数值，该参数值和具体的参数类型有关。

**1,GL_TEXTURE_MAG_FILTER**
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);
	
指当纹理图象被使用到一个大于它的形状上时（即：有可能纹理图象中的一个像素会被应用到实际绘制时的多个像素。例如将一幅256x256的纹理图象应用到一个512x512的正方形），应该如何处理。可选择的设置有GL_NEAREST和GL_LINEAR，前者表示“使用纹理中坐标最接近的一个像素的颜色作为需要绘制的像素颜色”，后者表示“使用纹理中坐标最接近的若干个颜色，通过加权平均算法得到需要绘制的像素颜色”。前者只经过简单比较，需要运算较少，可能速度较快，后者需要经过加权平均计算，其中涉及除法运算，可能速度较慢（但如果有专门的处理硬件，也可能两者速度相同）。从视觉效果上看，前者效果较差，在一些情况下锯齿现象明显，后者效果会较好（但如果纹理图象本身比较大，则两者在视觉效果上就会比较接近）。

	

**2,GL_TEXTURE_MIN_FILTER**
	
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
	
指当纹理图象被使用到一个小于（或等于）它的形状上时（即有可能纹理图象中的多个像素被应用到实际绘制时的一个像素。例如将一幅256x256的纹理图象应用到一个128x128的正方形），应该如何处理。可选择的设置有:<br>
GL_NEAREST,GL_LINEAR,GL_NEAREST_MIPMAP_NEAREST,GL_NEAREST_MIPMAP_LINEAR,GL_LINEAR_MIPMAP_NEAREST和GL_LINEAR_MIPMAP_LINEAR。其中后四个涉及到mipmap。前两个选项则和GL_TEXTURE_MAG_FILTER中的类似。此参数似乎是必须设置的（在我的计算机上，不设置此参数将得到错误的显示结果，但我目前并没有找到根据）。
	
	
	
**3,GL_TEXTURE_WRAP_S**
	
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_WRAP_S,GL_CLAMP);
			
指当纹理坐标的第一维坐标值大于1.0或小于0.0时，应该如何处理。<br>

GL_CLAMP:即超过1.0的按1.0处理，不足0.0的按0.0处理。<br>
GL_REPEAT:即对UV坐标值加上一个合适的整数（可以是正数或负数），得到一个在[0.0, 1.0]范围内的值，然后用这个值作为新的纹理坐标。
	

**4,GL_TEXTURE_WRAP_T**

与GL_TEXTURE_WRAP_S相似，指当纹理坐标的第二维坐标值大于1.0或小于0.0时，应该如何处理。


#其他:

glActiveTexture:<br>
<https://www.opengl.org/sdk/docs/man2/xhtml/glActiveTexture.xml>

#完整代码:

<https://github.com/sbxfc/OpenGLTextureMapping>

