---
layout: post
title: "OpenGL - 加载Obj文件"
date: 2014-10-20 14:19:21 +0800
comments: true
categories: 
---

obj是一种3D模型文件的格式,我们可以通过大部分建模工具将绘制的3D模型信息导出并保存成obj文件。因为obj是文本格式,所以我们很容易通过其规则将文件记录的模型信息读取出来,并进行加载和渲染。

以简单的立方体文件为例:(点击下面的连接)<br><http://people.sc.fsu.edu/~jburkardt/data/obj/cube.obj>
	
其中\#是注释部分,v表示顶点,vn表示法线,f是每个面的信息。其中每个面有多个顶点组成,一个立方体的一边是由两个三角形面拼成的。在示例中的,每个三角形面有如下信息:

	f  1//2  7//2  5//2

很显然,除了隐约知道第一个字母f是单词face的缩写以外,对于其他信息则是一脸茫然。这三组数据实际上记录了三个点的信息,obj格式里,每个面的格式 // 代表该标识的两侧,左边是一个顶点索引,右边是一个法线索引。所以,我们就知道了 1//2 标识,这个面的第一个顶点的顶点坐标索引是1,顶点法线的索引是2.该面就是一个由索引值为 1、7、5和法线索引为 2、2、2的值组成的三角形面。

至此,我们大致能读懂这个立方体的信息了。


	f v1//vn1 v2//vn2 v3//vn3 ...

每个面由不定数量的顶点组成。其中 v1//vn1 表示每个顶点包含一个顶点索引和法线索引。
相应的还有下面这种格式:

	f v1/vt1/vn1 v2/vt2/vn2 v3/vt3/vn3 ....
	
表示顶点由 顶点索引/UV点索引/法线索引 组成。

	f v1/vt1 v2/vt2 v3/vt3 ...
	
以及表示顶点由 顶点索引/UV点索引 组成的面。


obj Wiki:<br>
<http://en.wikipedia.org/wiki/Wavefront_.obj_file><br>
obj文件格式详解:<br>
<http://www.cppblog.com/lovedday/archive/2008/06/13/53153.html>


上面的立方体的有12个三角形面组成，但我们也可能遇到四边形或五边形。在函数`exportFaceGroupToShape`里,我们将其统一转成三角形。我们以组(group)的形式解析并绘制模型。

定义每个group的格式:

	typedef struct
    {
        std::vector<unsigned int>   indices;	  //顶点索引
        std::vector<float>          positions;  //顶点坐标
        std::vector<float>          normals;	  //顶点法线
        std::vector<float>          texcoords;	  //顶点的UV坐标
        std::vector<int>            material_ids;//顶点对应的材质信息
    } mesh_r;
	 
	typedef struct
    {
        std::string name;	//组名
        mesh_r       mesh;
    } shape_r;
    
indices包含了这个组所有顶点信息的索引。`positions` `normals` `texcoords` 和 `material_ids`分别是顶点索引对应的相应顶点信息。其中材质ID`material_ids`数组对应了每个顶点的材质信息。材质ID的索引对应的材质数组materials,它包含了所有顶点的材质信息，每个顶点通过ID去对应相应的材质信息。格式如下:

	typedef struct
    {
        std::string name;
        
        float ambient[3];
        float diffuse[3];
        float specular[3];
        float transmittance[3];
        float emission[3];
        float shininess;
        float ior;
        float dissolve;
        int illum;
        
        std::string ambient_texname;
        std::string diffuse_texname;
        std::string specular_texname;
        std::string normal_texname;
        std::map<std::string, std::string> unknown_parameter;
    } material_r;
    
OBJ文件的解析涉及到文件的读取和文件格式的转换,不详细解释。渲染的时候，我是根据每个组创建VAO然后绑定相应顶点数据。最后用一个循环的形式，通过`glDrawElements`将模型绘制出来。可能还有其他更好的办法，因为对渲染还不是很熟悉，暂时只用到这一种：
	
	for(int i = 0 ; i < shapersNum ;i++)
    {
        glGenVertexArrays(1,&m_vao[i]);
        glBindVertexArray(m_vao[i]);
        
            GLuint m_vbo[4];
            glGenBuffers(4, m_vbo);
            glBindBuffer(GL_ARRAY_BUFFER, m_vbo[0]);
            glBufferData(GL_ARRAY_BUFFER, shapes[i].mesh.positions.size()*sizeof(GLfloat), &shapes[i].mesh.positions[0], GL_STATIC_DRAW);
        
            glEnableVertexAttribArray(0);
            glVertexAttribPointer(0,3,GL_FLOAT,GL_FALSE,0,0);
        
            glBindBuffer(GL_ARRAY_BUFFER, m_vbo[1]);
            glBufferData(GL_ARRAY_BUFFER, shapes[i].mesh.normals.size()*sizeof(GLfloat), &shapes[i].mesh.normals[0], GL_STATIC_DRAW);
        
            glEnableVertexAttribArray(1);
            glVertexAttribPointer(1,3,GL_FLOAT,GL_FALSE,0,0);
        
            glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_vbo[3]);
            glBufferData(GL_ELEMENT_ARRAY_BUFFER, shapes[i].mesh.indices.size()*sizeof(GLuint), &shapes[i].mesh.indices[0], GL_STATIC_DRAW);

        glBindVertexArray(0);
    }

绘制代码:

	for(int i = 0 ; i < size ;i++)
    {
        glBindVertexArray(m_vao[i]);
        glDrawElements(GL_TRIANGLES,(GLsizei)shapes[i].mesh.indices.size(),GL_UNSIGNED_INT,0);
        glBindVertexArray(0);
    }


接下来将会将顶点的材质`material`绑定到VAO上，与传递顶点、法线的类似的形式，将材质信息传递到着色器。加载出来的没有贴图的汽车模型:

![obj](/images/2014/10/obj-car.png)

这里在网上找到的一个小的OBJ模型库:<br>
<http://people.sc.fsu.edu/~jburkardt/data/obj/obj.html>

tinyobjloader:<br>
<http://syoyo.github.io/tinyobjloader/>

#完整代码:
<https://github.com/sbxfc/OpenGLDrawOBJ>