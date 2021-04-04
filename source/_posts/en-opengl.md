# Introduce
OpenGL是一套标准，它定义了怎么使用显卡的API，Nvidia发行   
* 跨平台
* 简单
* 让我们可以更好得使用显卡，得到更好的画面
shader是运行在GPU上的代码  
<!--more-->

# Setup OpenGL and Create a Window in C++
跨平台  
但创建窗口是很依赖平台的，因此我们要使用一个三方库来为我们屏蔽平台之间的区别，即另一个抽象，这里我们用GLFW，因为它简单轻量  
首先从[glfw](https:://www.glfw.org)下载二进制包
* 把头文件和库复制到`&(SolutiionDir)Dependencies`下
* 在设置的常规里包含额外头文件路径
* 在设置的链接器里包含头文件路径
* 链接器部分包含静态库文件夹，以及在输入文件里包含静态库文件
* 在预处理器定定义里包含某些预定义宏

# Using Modern OpenGL in C++


# Vertex Buffers and Draw a Triangle
使用现代OpenGL需要创建一些Buffer和自己的shader  
Vertex Buffer是一个Buffer，在GPU显存里的内存  
通过`void glGenBuffers(int nums, int* bufferId);`来生成一个buffer，把这个buffer的id存在bufferId里，这个id是一个无符号整数