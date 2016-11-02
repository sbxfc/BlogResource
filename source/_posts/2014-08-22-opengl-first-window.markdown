---
layout: post
title: "OpenGL - 使用glfw库创建新窗口"
date: 2014-08-22 17:39:39 +0800
comments: true
categories: 
---

#下载&安装(macOS)

一, 下载glfw库(当前版本3.0.4): 

- <http://www.glfw.org/download.html>

二, 安装:
	
	$ cd /下载目录/glfw-master
	$ cmake .
	$ sudo make install

在编译完成后,前往目录(CMD+Shift+G) /usr/local/,可以看到生成的 lib 和 include 文件内容


#配置&运行示例

一, 用 Xcode 新建一个 Command Line Tool 项目。

二, 选择右侧TARGET,在 Build Phases 里的 Link Binary With Libraries 添加 Cocoa.framework 、 CoreVideo.framework 、IOKit.framework 、OpenGL.framework,在 Add Other里按 CMD+Shift+G 添加 /usr/local/lib 目录下的 libglfw3.a

三. 在 Build Settings 里设置搜索路径,点 Header Search Path 添加 /usr/local/include,接着设置lib文件路径,在 Library Search Path 里添加 /usr/local/lib


四,打开main.cpp,将以下示例代码拷贝至main.cpp中,按 Command + R 运行示例:

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


#截屏



![glfw](/images/2014/8/tmp5812c0ca.png)



