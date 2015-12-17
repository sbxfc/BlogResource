---
layout: post
title: "Unity-正交、透视相机原理"
date: 2014-05-12 16:46:05 +0800
comments: true
categories: unity
categories: 
---

一,透视投影相机 (Perspective Camera)

---

使用透视相机获取的景物类似人眼中的真实世界,有“近大远小”的效果。

![](/images/2014/5/perspective-camera.jpg)


在这张透视照相机投影图中，灰色的部分是视景体，是可能被渲染的物体所在的区域。fov是视景体竖直方向上的张角，如侧视图所示。

照相机水平方向和竖直方向长度的比值（width / height），通常设为视景体的横纵比例。

`near`和`far`分别是照相机到视景体最近、最远的距离，均为正值，且`far`应大于`near`。

透视投影矩阵就是根据这些值计算出的,下面是一段OpenGL里生成投影矩阵的代码。

	glm::mat4 projection = glm::perspective(fov, width/height, near, far);

<!--more-->
<br><br>
二,正交投影相机 (Orthographic Camera)

---

正交相机获得的景象，类似几何课上黑板上画的效果。在三维空间内平行的线，投影到二维空间中也一定是平行的.

![按实际比例显示景物，没有变形](/images/2014/5/orthographic.png)

![](/images/2014/5/orthographic-camera.jpg)

图中,`left`, `right`, `top`, `bottom`, `near`, `far`,这六个参数分别代表正交投影照相机拍摄到的空间的六个面的位置，这两个面围成一个长方体，我们称其为视景体（Frustum）。只有在视景体内部（下图中的灰色部分）的物体才可能显示在屏幕上，而视景体外的物体会在显示之前被裁减掉。

其中,照相机的横竖比例确定，(`right` - `left`)与(`top` - `bottom`)的比例与视景体宽度与高度的比例一致。

`near`与`far`都是指到照相机位置在深度平面的位置，而照相机不应该拍摄到其后方的物体，因此这两个值应该均为正值。为了保证场景中的物体不会因为太近或太远而被照相机忽略，一般`near`的值设置得较小，`far`的值设置得较大，具体值视场景中物体的位置等决定。

<br>


三,正交相机和透视相机的选择

---

一般说来，对于制图、建模软件通常使用正交投影，这样不会因为投影而改变物体比例；<br>对于其他大多数应用，通常使用透视投影，因为这更接近人眼的观察效果。

<br><br>
OpenGL- 透视矩阵:<br>
<http://rungame.me/blog/2014/09/08/gl-projection-matrices/>