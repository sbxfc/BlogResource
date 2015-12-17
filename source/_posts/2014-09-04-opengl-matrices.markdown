---
layout: post
title: "OpenGL - 矩阵转换"
date: 2014-09-04 10:03:08 +0800
comments: true
categories: 
---



矩阵与点的乘法公式:

![](/images/2014/9/MatrixXVect.gif)

#一,平移
平移矩阵是最简单的矩阵。其中X、Y、Z 代表位移的增量

![](/images/2014/9/translationMatrix.png)


例如,将点(10,10,10) 向x轴正方向移动十个单位，可得:

![](/images/2014/9/translationExamplePosition1.png)

用代码表示:

	glm::vec4 vector(10,10,10,0);
	glm::mat4 translateMatrix = glm::translate(10,0,0);
	glm::vec4 transformedVector = translateMatrix * vector; 

<br>	
#二,缩放 

缩放矩阵和平移矩阵类似,图中X、Y、Z分别代表沿X、Y、Z轴的缩放值。

![](/images/2014/9/scalingMatrix.png)

把一个点沿各个方向放大2倍。

![](/images/2014/9/scalingExample.png)

用代码缩放:
	
	glm::vec4 vector(10,10,10,0);
	glm::mat4 scalingMatrix = glm::scale(2,2,2);
	glm::vec4 transformedVector = scalingMatrix * vector; 

PS:若缩放值为负，则沿着坐标轴的反方向进行缩放。

#三,旋转

沿Y轴45度的旋转:

	glm::vec3 rotationAxis(0,1,0);
	glm::rotate(rorateY, 45.0f, rotationAxis);
	
将旋转,缩放,平移组合起来只需要将他们相乘即可,比如:

	TransformedVector = TranslationMatrix * RotationMatrix * ScaleMatrix * OriginalVector;
	
注意,不同的相乘顺序得到的结果是不同的。
<br>
代码里绘制了一个正方形,通过'A' 'D'键控制它旋转。大部分跟之前的代码是一致的。

#使用glm库

<http://rungame.me/blog/2015/10/29/glm/>

#完整代码

<https://github.com/sbxfc/OpenGLMatrices>