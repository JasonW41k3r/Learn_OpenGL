# Learn OpenGL
## 入门
### What's OpenGL?
OpenGL是由Khronos组织制定并维护的规范，规定了OpenGL的API中的函数应该如何执行，以及他们的输出值。OpenGL API包含了一系列可以操作图像，图形的函数。  
OpenGL只规定了函数的执行方式以及输出，并不规定函数的具体实现方式。函数的具体实现细节由OpenGL库的开发者自己决定（通常是显卡的生产商）。

#### 核心模式和立即渲染模式
早期OpenGL使用立即渲染模式。OpenGL的立即渲染模式是传统的“固定功能管线”，简单易用，但灵活性差，而核心模式是现代的“可编程管线”，需要自己编写着色器，虽然复杂但更高效、灵活，适合现代图形开发。当使用已经废弃的函数时，OpenGL会抛出错误并终止绘图。

#### 扩展
扩展是不包含在Khronos官方OpenGL规范当中的功能实现。显卡公司通常以扩展的方式提供新特性和渲染优化。  
如果一个扩展非常流行或非常有效，可能会成为OpenGL未来规范的一部分。扩展的源码通常看上去如下：
```c
if(GL_ARB_extension_name)
{
    // 使用硬件支持的全新的现代特性
}
else
{
    // 不支持此扩展: 用旧的方式去做
}
```

#### 状态机
OpenGL就是一个硕大的状态机。我们通过变量描述OpenGL状态（上下文）来告诉OpenGL应该如何渲染。

#### 对象
OpenGL库使用**C语言**进行编写。OpenGL引入抽象层“对象”来表示OpenGL状态的一个子集，每个对象都表示了OpenGL的部分状态。
```c
struct object_name {
    float  option1;
    int    option2;
    char[] name;
};
```

### 创建窗口
在GLFW当中，小写字母开头的通常是函数名，而大写字母开头的通常是类型名。  
创建窗口的代码通常如下：
```c++
#include <glad/glad.h>  // GLAD用于获取系统的OpenGL相关函数指针
#include <GLFW/glfw3.h> // GLFW用于管理OpenGL窗口的上下文

int main(void)
{
    glfwInit();         // 初始化GLFW
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);  // 设置OpenGL主版本号
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);  // 设置OpenGL次版本号
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);  // 设置OpenGL的配置文件为核心模式
    ...
}
```
`glfwWindowHint()`函数用于配置GLFW。其中第一个参数是选项的名称，第二个参数是选项的值。  

**注意**：在引入GLFW库文件之前，需要先引入GLAD库文件，因为GLAD库文件中包含了OpenGL的所有函数指针。如果不先引入GLAD库文件，GLFW库文件就会优先使用系统自带的OpenGL库，而不是GLAD库文件中的OpenGL库，从而导致系统库和GLAD库冲突。

接下来创建窗口对象。代码如下所示：
```c++
GLFWwindow* window = glfwCreateWindow(800, 600, 'LearnOpenGL', NULL, NULL);
glfwMakeContextCurrent(window);
```
`glfwCreateWindow()`函数的作用是创建一个GLFW的窗口对象，类型是`GLFWwindow*`指针类型。其中第一个参数是宽度，第二个参数是高度，第三个参数是窗口名称。忽略第四个和第五个参数。  

`glfwMakeContextCurrent()`函数接受一个窗口对象，作用是将当前的线程的GLFW上下文和参数的窗口对象绑定。

#### GLAD
GLAD用于管理OpenGL的函数指针，所以调用任何OpenGL函数之前都需要初始化GLAD。以下代码用于初始化GLAD：
```c++
gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);
```
`glfwGetProcAddress()`是一个函数，用于根据上下文获取OpenGL函数的地址，而`gladLoadGLLoader()`函数的作用是初始化所有在`glfwGetProcAddress()`中可以获取到的函数地址。等于执行了所有类似以下的代码：
```c++
glClear = (PFNGLCLEARPROC)glfwGetProcAddress("glClear");
glDrawArrays = (PFNGLDRAWARRAYSPROC)glfwGetProcAddress("glDrawArrays");
glBindBuffer = (PFNGLBINDBUFFERPROC)glfwGetProcAddress("glBindBuffer");
// 以及所有其他 OpenGL 函数
```
接下来我们就可以使用以`gl...()`开头的OpenGL函数了。

#### 视口
我们必须在渲染前告诉OpenGL渲染窗口的尺寸大小，即视口（viewport）。视口针对渲染而言，而窗口是显示内容的所有区域，包括渲染内容和其他UI内容。
```c++
glViewport(0, 0, 800, 600);
```
前两个参数控制**左下角**的位置，后两个参数控制视口的宽度和高度。

#### 回调函数（callback function）
回调函数是在特定条件被触发时会执行的函数。我们可以设置回调函数，用于在用户更改窗口大小的时候自动更改视口大小。首先我们需要写一个回调函数：
```c++
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    glViewport(0, 0, width, height);
}
```
接下来我们需要在`main()`函数当中将该函数向OpenGL注册，从而OpenGL知道条件触发时（比如用户修改窗口大小）应该执行什么内容：
```c++
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```
用户更改窗口大小的时候，OpenGL会自动调用`framebuffer_size_callback()`函数并填入相关的参数（比如宽度和高度），并根据用户定义的函数体执行相关操作。