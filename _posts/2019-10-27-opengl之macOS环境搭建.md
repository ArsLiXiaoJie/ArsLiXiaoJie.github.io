---
layout: post
title:  "openGL 之macOS环境搭建"
date:   2019-10-27 15:00:00 +0800
categories: opengl
tag: opengl
typora-root-url: ../../arslixiaojie.github.io
---

​	参考 [点击](http://blog.shenyuanluo.com/OpenGLEnvironment.html)

# 工具准备

1、Xcode

2、Command Line Tool

3、[CMake](https://cmake.org/download/)



# 简述

- **OpenGL**：（Open Graphics Library）是用于渲染2D、3D 矢量图形的跨语言、跨平台（由 Khronos组织 制定并维护）的规范(Specification)。
- **GLFW**：是一个专门针对 OpenGL 的 C 语言库，提供了渲染物体所需的最低限度的接口。其允许用户创建 OpenGL 上下文，定义窗口参数以及处理用户输入，把物体渲染到屏幕所需的必要功能。（注意：OpenGL 并不规定窗口创建和管理的部分，这一部分完全交由 GLFW 来实现；还有其他类似的：GLUT 和 SDL 等）。
- **GLEW**：由于 OpenGL 只是一种 标准/规范，并且是由驱动制造商在驱动中予以实现。OpenGL 的大多数函数在编译时（compile-time）是未知状态的，需要在运行时（run-time）来请求。GLEW 的工作就是获取所需的函数的地址，并储存在函数指针中供使用。（还有其他类似的：GLAD）。
- **GLAD**：是一个开源的库，功能跟 GLEW 类似。GLAD 使用了一个在线服务（在这里能够告诉 GLAD 需要定义的 OpenGL 版本，并且根据这个版本加载所有相关的 OpenGL 函数）。



# GLFW编译安装

1、下载[GLFW源码](https://www.glfw.org/download.html)，并解压

2、打开CMake，源代码目录选择刚解压出来的源码目录，目标目录选择输出目录。

<img src="/assets/opengl_macos1.png" alt="1" style="zoom:50%;" />

3、点击Configure按钮，让CMake读取设置和源代码，然后在弹出的界面中选择工程的生成器 Unix Makefiles。

<img src="/assets/opengl_macos2.png" alt="2" style="zoom:50%;" />

4、CMake会显示可选的编译选项来配置最终生成的库。这里使用默认配置，并再次点击Configure按钮以保存设置。

<img src="/assets/opengl_macos3.png" alt="3" style="zoom:50%;" />

5、点击 Generate 按钮，生成的工程文件会保存到指定的输出目录中。

<img src="/assets/opengl_macos4.png" alt="4" style="zoom:50%;" />

6、打开终端，进入到此目录中，执行

```bash
make
```

<img src="/assets/opengl_macos5.png" alt="5" style="zoom:50%;" />

7、执行 make install 命令进行安装，一般会安装到 /usr/local/include 和 /usr/local/lib 中。

```bash
make install
```

如果提示说没有权限，则在前面加个 sudo

8、可以在 /usr/local/include 文件夹中看到生成了一个 GLFW 文件夹，在 /usr/local/lib 文件夹中有 libglfw3.a 。

至此，macOS中的GLFW环境已经配置好了。



注意：

glew头文件必须包含在glfw头文件前面，否则会报错。



# GLEW编译安装

1、下载[GLEW源码](http://glew.sourceforge.net/)，并解压

2、打开终端，进入到解压目录

3、执行 make 命令进行编译安装库文件

4、执行 make install 命令进行安装，一般会安装到 /usr/local/include 和 /usr/local/lib 中（没权限就加sudo）

5、可以看到 /usr/local/include 中多了一个 GL 文件夹，在 /usr/local/lib 中多了几个 GLEW 的库。

至此，macOS中的GLEW环境已经配置好了。



# GLAD配置

1、打开GLAD的 [在线服务](https://glad.dav1d.de/)

2、将语言设置为 C/C++，在API选项中，选择3.3及以上的OpenGL版本

3、将模式Profile设置为 Core，保证生成加载器 Generate a loader选项是选中的

4、扩展Extensions中的内容暂时忽略

5、点击生成 Generate

<img src="/assets/opengl_macos6.png" alt="6" style="zoom:50%;" />

6、下载生成的zip包，（包含glad.c、glad.h和khrplatform.h），解压后include目录下的glad和KHR文件夹复制到 /usr/local/include ，glad.c添加到项目中。



注意：

glad头文件必须包含在glfw头文件之前，否则会报错，而且不能和glew共同使用，否则也会报错。





# 使用

要在Xcode中使用相应的库函数，需要在Xcode中添加相应的头文件和库文件，一般有两种方法。

1. 将库的include和lib文件夹拷贝到Xcode本身到include和lib文件夹中（不推荐）。
2. 在新建的工程中链接库的include和lib文件。

步骤：

1. 在工程的 Build Settings 中，找到 Header Search Paths，将 /usr/local/include 添加到头文件搜索路径中，记得选上 recursive。

2. 在工程的 Build Settings 中，找到 Library Search Paths，将 /usr/local/lib 添加到库文件搜索路径中，记得选上 recursive。

3. <img src="/assets/opengl_macos7.png" alt="7" style="zoom:50%;" />

4. 在工程的 Build Phases 中的 Link Binary With Libraries 中，添加以下的库

   Cocoa.framework

   OpenGL.framework

   GLUT.framework

   CoreVideo.framework

   IOKit.framework

   libglfw.a (需手动导入)

   libGLEW.a (需手动导入)

5. 把 Build Settings 中的 Enabled Modules(C and Objective-C) 设为NO，否则GLEW头文件会报错。





# 测试

1. 新建一个 Command Line Tool 工程。
2. 配置环境
3. 输入以下测试代码，会看到一个三角形

```c++
#include <GL/glew.h>
#include <GLFW/glfw3.h>
void Render(void)
{
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);
    glBegin(GL_TRIANGLES);
    {
        glColor3f(1.0,0.0,0.0);
        glVertex2f(0, .5);
        glColor3f(0.0,1.0,0.0);
        glVertex2f(-.5,-.5);
        glColor3f(0.0, 0.0, 1.0);
        glVertex2f(.5, -.5);
    }
    glEnd();
}
int main(int argc, const char * argv[]) {
    GLFWwindow* win;
    if(!glfwInit()){
        return -1;
    }
    win = glfwCreateWindow(640, 480, "OpenGL Base Project", NULL, NULL);
    if(!win)
    {
        glfwTerminate();
        return -1;
    }
    if(!glewInit())
    {
        return -1;
    }
    glfwMakeContextCurrent(win);
    while(!glfwWindowShouldClose(win)){
        Render();
        glfwSwapBuffers(win);
        glfwPollEvents();
    }
    glfwTerminate();
    return 0;
}

```





