---
layout: post
title: "OpenGL - 绘制三角形"
date: 2014-09-01 09:45:58 +0800
comments: true
categories: 
---

OpenGL是一个跨平台的图形绘制接口。通过它,开发者可以使用GPU的高速处理能力来绘制图像。

在OpenGL里,三角形是最基本的图形单元。我们将通过绘制三角形来了解OpenGL的工作流程,并利用[GLFW库](http://rungame.me/blog/2014/08/22/opengl-first-window/)的跨平台环境将三角形显示在窗口上。

#VBO

CPU和GPU处在两个不同的内存单元。在通过GPU绘制之前,我们首先在本地创建一份顶点数据:

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

在这两条数据里,我们设置了三角形的顶点坐标和颜色。这两份数据最终会通过OpenGL接口传递到显卡上,并在显卡上生成一份成为VBO(顶点缓冲对象)的副本,实际上就是一块二进制内存。着色器在渲染时才需要为GPU指定VBO的具体格式,所以我们只是简单地创建相应的VBO对象并上传顶点数据到VBO对象上:

	glGenBuffers(2, vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo[0]);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertexPosition), vertexPosition, GL_STATIC_DRAW); 
   
OpenGL和其他语言的习惯不同,OpenGL使用一个无符号的整数作为对象的唯一标识,每个对象都对应一个唯一标识。该标识可以通过[glGenBuffers](https://www.khronos.org/opengles/sdk/docs/man32/html/glGenBuffers.xhtml/)函数由系统自动生成,也可以自己手动随便指定一个无符号的整数值,这两者的区别是前者能保证唯一性。

通过glBindBuffer函数,我们为缓冲对象标识初始化一个GL_ARRAY_BUFFER类型的缓冲对象,即顶点缓冲对象VBO。如果该唯一标识对应的顶点缓冲对象已经存在,glBindBuffer还有选择或激活该缓冲区的意思,即在绑定之后未来的操作将会影响该缓冲区。
		
最后,使用glBufferData将本地的顶点数据提交至顶点缓冲区。
	
glBufferData的结构:
	
	void glBufferData(	GLenum target,
 						GLsizeiptr size,
 						const GLvoid * data,
 						GLenum usage);
	
- target : 对象类型
- size : 对象的字节长度
- data : 数据 (<font color="#bd260d">**这里设置的是本地顶点数据的指针,并且上传到显卡内存的是该数据的副本(copy的形式)**</font>)
- usage : 使用方式。在我们的程序中顶点数组数据都是常量,不需要改变,因此我们给glBufferData一个GL_STATIC_DRAW的参数。其中STATIC部分表明我们不会想去改变数据,也可以设置为DYNAMIC,表明我们频繁地写到这个缓冲里,或者STREAM,表明我们将周期性地替换掉缓冲的内容。DRAW部分表明我们希望缓冲只会被GPU读取。与DRAW相对的是READ,表明一个缓冲还要被CPU读回去。还有COPY,表明这个缓冲是CPU和GPU之间的一个管道,不应该偏重于任一方。顶点数组几乎总是使用GL_*_DRAW表示。

调用glBufferData函数为VBO指定数据时,你可以为该对象指明相应的数据格式、大小和使用方式。如果你设置的使用方式和你在之后的处理方式不同也不会出错,但会造成效率低下。glBufferData处理数据的方式很类似memcpy,仅仅是一串没有特别意义的字节流。直到我们渲染它之前，我们不会告诉OpenGL我们数组的结构。这允许缓冲以几乎任何格式存储顶点属性以及其它数据，或者同一份数据给不同的渲染任务以不同的方式去处理。 

在glBufferData函数调用之后，数据会上传到显卡缓存里。如果你不再需要对顶点做修改、重新上传的话，在这里可以将本地内存里的该顶点数据释放掉。

#VAO 

从本质说,VBO相当于显卡上一块记录顶点数据的Buffer,要想绘制一个三角形除了顶点数据以外,在渲染时我们还要告诉GPU顶点的格式和使用情况以及vertex-shader里的属性对应的顶点数据位置。以上的这些就是VAO包含的内容,VAO记录了一次绘制所需的所有顶点信息。

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


#完整代码:

完整工程:<br>
<https://github.com/sbxfc/OpenGLDrawTriangle>

#参考:

<http://stackoverflow.com/questions/18368914/gldrawarrays-behaving-weirdly-on-mac-os-x>

<http://www.cnblogs.com/mikewolf2002/archive/2012/10/27/2742137.html>