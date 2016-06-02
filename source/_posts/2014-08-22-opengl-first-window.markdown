---
layout: post
title: "OpenGL - 使用glfw库创建窗口"
date: 2014-08-22 17:39:39 +0800
comments: true
categories: 
---

#下载glfw库

当前版本3.0.4: 

- <http://www.glfw.org/download.html>

#Mac下安装
	
	$ cd /下载目录/glfw-master
	$ cmake .
	$ sudo make install

在完成编译之后,(CMD+Shift+G)前往目录 /usr/local/,查看生成的 lib 和 include 文件内容


#新建glfw窗口

1. 用Xcode新建一个Command Line Tool项目

2. 在Build Phases里的Link Binary With Libraries添加 Cocoa.framework 、 CoreVideo.framework 、IOKit.framework 、OpenGL.framework,在添加其他里按CMD+Shift+G 添加 /usr/local/lib 目录下的 libglfw3.a

3. 在 Build Settings 里设置头文件的搜索路径,点 Header Search Path 添加 /usr/local/include,接着设置lib文件路径,在 Library Search Path 里添加 /usr/local/lib


#运行测试示例


下面的代码是从glfw官网复制来的代码,将下面代码copy到main.m文件里,直接Command + R运行

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
	
#完整示例

- <https://github.com/sbxfc/glfw-window>