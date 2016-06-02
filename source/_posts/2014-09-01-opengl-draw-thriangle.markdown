---
layout: post
title: "OpenGL - 绘制三角形"
date: 2014-09-01 09:45:58 +0800
comments: true
categories: 
---


[glfw窗口创建之后](http://rungame.me/blog/2014/08/22/opengl-first-window/),我们就可以利用它,将要渲染的三角形显示在屏幕上。

#绘制三角形

首先,创建本地的顶点坐标和顶点颜色数据。
	
	//三角形的顶点坐标
    GLfloat vertexPosition[] = {
        -1.0f,-1.0f,0.0f, //第一个顶点坐标
        1.0f,-1.0f,0.0f,  //第二个顶点坐标
        0.0f,1.0f,0.0f,   //第三个顶点坐标
    };
    
    //三角形的顶点颜色
    GLfloat vertexColor[] = {
        1.0, 0.0, 0.0,  //第一个顶点颜色的RGB值
        0.0, 1.0, 0.0,  //第二个顶点颜色的RGB值
        0.0, 0.0, 1.0,  //第三个顶点颜色的RGB值
    };
    
接着,创建这两份数据的在显卡上的副本(VBO)。

	glGenBuffers(2, vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo[0]);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertexPosition), vertexPosition, GL_STATIC_DRAW);

glGemBuffers函数用来生成一个唯一的无符号整数作为VBO标识,我们每次操作VBO都要通过该标识。你可以自己指定一个VBO标识,但手动指定的标识无法保证唯一性。

顶点坐标和颜色通过 glBufferData 函数上传至显卡。在此之前,你需要通过 glBindBuffer 函数绑定你想要操作的VBO对象。如果 glBindBuffer 指定的标识是第一次被绑定,该函数还有初始化缓存对象的作用。

绑定之后,我们就可以使用glBufferData函数将顶点数据提交至顶点缓冲区。glBufferData 要比以上两个函数要复杂一点。
	
	void glBufferData(	
				GLenum target,
				GLsizeiptr size,
				const GLvoid * data,
				GLenum usage);
	
- target : 对象类型
- size : 对象的字节长度
- data : 数据 (<font color="#bd260d">**这里设置的是本地顶点数据的指针,并且上传到显卡内存的是该数据的副本(copy的形式)**</font>)
- usage : 使用方式。在我们的程序中,顶点数组数据都是常量,不需要改变,因此我们设置了一个 GL_STATIC_DRAW 类型。其中STATIC部分表明我们不会去改变数据,也可以设置为DYNAMIC,表明我们频繁地写到这个缓冲里,或者STREAM,表明我们将周期性地替换掉缓冲的内容。DRAW部分表明我们希望缓冲只会被GPU读取。与DRAW相对的是READ,表明一个缓冲还要被CPU读回去。还有COPY,表明这个缓冲是CPU和GPU之间的一个管道,不应该偏重于任一方。顶点数组几乎总是使用GL_*_DRAW表示。

调用glBufferData函数为VBO指定数据时,你可以为该对象指明相应的数据格式、大小和使用方式。假如你设置的使用方式和你在之后的处理方式不同也不会出错,但会造成效率低下。

glBufferData处理数据的方式很类似memcpy,仅仅是一串没有特别意义的字节流。直到我们渲染它之前，我们不会告诉OpenGL我们数组的结构。这允许缓冲以几乎任何格式存储顶点属性以及其它数据，或者同一份数据给不同的渲染任务以不同的方式去处理。 

在glBufferData函数调用之后，数据会上传到显卡缓存里。如果你不再需要对顶点做修改、重新上传的话，在这里可以将本地内存里的该顶点数据释放掉。

#VAO 

要想绘制一个三角形除了顶点数据以外,在渲染时我们还要告诉GPU顶点的格式和使用情况以及vertex-shader里的属性对应的顶点数据位置。以上的这些就是VAO包含的内容,VAO包含了一次绘制所需的全部顶点信息。

使用VAO的好处是,我们可以在初始化时一次性讲GPU所需要的绘制信息(包括顶点数据和使用情况)上传至显卡,在每帧绘制时不必重复传递顶点数据和设置顶点的数据结构等,减少了CPU和GPU之间的交互。

创建一个VAO对象和VBO是一样的:

	glGenVertexArrays(1,&vao);
    glBindVertexArray(vao);

在绑定完VAO之后,接下来的一系列设置都会被记录到VAO中。首先,我们要为着色器设置渲染时着色器属性对应的顶点数据格式以及位置:

	glBindBuffer(GL_ARRAY_BUFFER, vbo[0]);
	glVertexAttribPointer(ATTRIB_LOCATION_VERTEX,3,GL_FLOAT,GL_FALSE,0,0);

在渲染时,一个VBO的值被解释为一个位置,着色器通过 layout (location = 0) 来关联顶点属性,然后使用glVertexAttribPointer将缓存中的VBO的位置与顶点着色器里的顶点的位置绑定在一起。
	
默认情况下,着色器里的属性访问都是关闭的,我们需要根据该属性对应的(location)通过glEnableVertexAttribArray将其设置为可访问。

	glEnableVertexAttribArray(ATTRIB_LOCATION_VERTEX);
		
最后,在绑定结束时需要显示地调用glBindVertexArray(0)来关闭VAO


在介绍VBO时,我们提到过,向VBO设置数据时我们没有告诉GPU缓存里VBO的具体格式。但在设置VAO时,我们通过glVertexAttribPointer函数将VBO和shader里的对应顶点渲染属性关联在一起,并在此时指定了VBO的格式。
	

*另外,我们也可以在 shader 里简单地声明顶点属性,比如:* 

	in vec3 position;

*在程序中glGetAttribLocation函数在运行时查找该属性位置信息,通过该函数返回的位置将其与VBO绑定在一起。这样就省去了硬编码的烦恼。*

ps. [glVertexAttribPointer要在绑定VBO后设置](https://www.khronos.org/opengles/sdk/docs/man/xhtml/glVertexAttribPointer.xml)
 
#绘制三角形
 
调用glDrawArrays函数将当前缓冲区里的顶点数据逐一进行绘制,因为GPU采用并行设计所以其绘制速度是很快的

	glDrawArrays(GL_TRIANGLES, 0, 3);
	
函数结构:

	void glDrawArrays(GLenum mode,
 					   GLint first,
 					   GLsizei count);

**参数**
 	
一,mode:
	
指定绘制图元的类型

- GL_POINTS : 绘制一个个点。
- GL_LINE_STRIP : 绘制两点之间的连线,第一个点和最后一个点不连接在一起。
- GL_LINE_LOOP : 绘制两点之间的连线,第一个点和最后一个点连接。
- GL_LINES : 以两点为一个组合连线。任意组合与其他的组合无联系,即绘制一条条线段。	
- GL_TRIANGLE_STRIP : 以V0V1V2,V1V2V3,V2V3V4,.....	
- GL_TRIANGLE_FAN : 以V0V1V2,V0V2V3,V0V3V4,....的顺序绘制三角形,类似于扇子的形状。
- GL_TRIANGLES : 每三个顶点之间绘制三角形,与其他三角形不连接。

二,first

在顶点数组里,第一个绘制的顶点索引。

三,count

指定绘制的顶点数量。


#完整示例

- <https://github.com/sbxfc/opengl/tree/master/opengl-draw-thriangle/demo>

#参见

- <https://www.khronos.org/opengles/sdk/docs/man32/html/glGenBuffers.xhtml/>

- <http://stackoverflow.com/questions/18368914/gldrawarrays-behaving-weirdly-on-mac-os-x>

- <http://www.cnblogs.com/mikewolf2002/archive/2012/10/27/2742137.html>