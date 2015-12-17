---
layout: post
title: "OpenGL - 在Xcode里使用GLFW"
date: 2014-08-22 17:39:39 +0800
comments: true
categories: 
---

#下载


当前版本是3.0.4: 

- <http://www.glfw.org/download.html>

#安装

	$ cd /下载路径/Downloads/glfw-master
	$ cmake .
	$ sudo make install

编译完成后,前往(Cmd+Shift+G)<font color="#bd260d">**/usr/local/**</font>查看生成的lib和include文件内容<br>

#新建GLFW

1. 用Xcode新建一个Command Line Tool项目

2. 在Build Phases->Link Binary With Libraries里添加 Cocoa.framework 、 CoreVideo.framework、IOKit.framework 、OpenGL.framework,在其他里按Cmd+Shift+G添加/usr/local/lib目录下的libglfw3.a

3. 在Build Settings里设置头文件路径,点Header Search Path 添加 /usr/local/include,然后设置库路径,在Library Search Path里添加 /usr/local/lib


#运行


下面的代码是从GLFW官网复制的,将代码copy到main.m文件里,直接Command + R运行

	#include <GLFW/glfw3.h>

	int main(void)
	{
	    GLFWwindow* window;
	
	    /* Initialize the library */
	    if (!glfwInit())
	        return -1;
	
	    /* Create a windowed mode window and its OpenGL context */
	    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
	    if (!window)
	    {
	        glfwTerminate();
	        return -1;
	    }
	
	    /* Make the window's context current */
	    glfwMakeContextCurrent(window);
	
	    /* Loop until the user closes the window */
	    while (!glfwWindowShouldClose(window))
	    {
	        /* Render here */
	
	        /* Swap front and back buffers */
	        glfwSwapBuffers(window);
	
	        /* Poll for and process events */
	        glfwPollEvents();
	    }
	
	    glfwTerminate();
	    return 0;
	}
	
#完整工程

<https://github.com/sbxfc/GLFWDemo>
