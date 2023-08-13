---
title: "1、你好，三角形"
weight: 2
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---


## 绘制之前。。
    
在绘制三角形或其它图形之前，最先想到的是创建一个窗口，来显示图形。

但在OpenGL中，这个过程比较复杂。OpenGL是由一系列变量描述当前如何操作的巨大状态机。这个状态通常被称为OpenGL上下文。一般通过设置选项、操作缓冲更改OpenGL状态。最后使用这个上下文进行渲染。

所以在一切开始之前，首先要**创建一个OpenGL上下文和一个窗口。**

然而这又会遇到一些问题，这些操作在每个系统上是不一样的。~~因为OpengL真的很抽象~~意味着很多东西要从底层开始，
这时就要用到一些针对OpenGL的库，这里使用的是GLFW。

同时又又又又因为，OpenGL驱动版本非常多，里面有很多函数非常乱，需要在运行时查询。
对于开发者而言则需要在运行时获取函数地址并存在某个指针中方便以后调用。为了简化这过程，需要用到GLAD库。


## GLFW

之后的所有操作中都要用到GLFW，初始化GLFW后，首先告诉GLFW使用opengl的版本，以及工作模式。

```c++
//初始化
glfwInit(); 
//opengl 版本为3.3
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
//核心模式
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);  
```

接下来就可以定义窗口了，定义窗口大小为800*600，标题为LearnOpenGL，
同时告诉GLFW将window设为当前线程主要上下文。
```c++
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
if (window == NULL)
{
    std::cout << "Failed to create GLFW window" << std::endl;
    glfwTerminate();
    return -1;
}
glfwMakeContextCurrent(window);
```

## GLAD

这会可以开始初始化GLAD，不然之后调用函数会报错。

```c++
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
{
    std::cout << "Failed to initialize GLAD" << std::endl;
    return -1;
}
```

## 视口

有了显示窗口还不太够，还必须告诉OpenGL渲染窗口的大小，即视口（Viewport）。
这样OpenGL就可以根据窗口大小显示数据和坐标。
```c++
glViewport(0, 0, 800, 600);
```
前面两个参数控制窗口左下角的位置，后两个参数控制大小（单位为像素）


一切完成后，再使用一个循环让窗口一直显示，这被称为渲染循环。
```c++
//渲染循环
while(!glfwWindowShouldClose(window)) //判断window是否被要求退出
{
    glfwSwapBuffers(window);    //交换颜色缓冲，绘制图形（存储window中每个像素颜色值的缓冲）
    glfwPollEvents();    //检查触发时间
}

```

最后释放分配的资源
```c++
glfwTerminate();
```

此时便完成了窗口的显示，编译运行后可以看到一个黑色的窗口，也可以更改窗口颜色
```c++
//渲染循环
while(!glfwWindowShouldClose(window)) //判断window是否被要求退出
{
    //输入
    //...

    //渲染
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);   //改颜色
    glClear(GL_COLOR_BUFFER_BIT);   //清空颜色缓冲，也有其它缓冲如depth buffer、stencil buffer

    //检查并调用事件
    glfwSwapBuffers(window);    //交换颜色缓冲，绘制图形（存储window中每个像素颜色值的缓冲）
    glfwPollEvents();    //检查触发时间
}
glfwTerminate();
```

## OpenGL渲染管线

我们最终的目的是为了绘制三角形，现在已经有了窗口，但是距离渲染绘制仍有一段距离。

从所周知，我们生活的世界是三维的，但我们看的屏幕却是二维的。
在OpenGL中，任何事物也都是3D的。为了将3D坐标转化为显示在窗口上2D坐标，这个过程称为图形渲染管线（Graphics Pipeline）。

图形管线可以分为两个主要部分：
- 将3D坐标转为2D坐标
- 将2D坐标转为有颜色的像素 [^1]
[^1]: 2D坐标和像素是不同的，前者表示在2D空间中的精确位置，后者只是该点的近似，受到了屏幕/窗口分辨率的限制

关于OpenGL的图形渲染管线可以参考下图

![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/pipeline.png)

可以看到在不同的阶段上都有一个各自运行的小程序，这些小程序被称为着色器，它们都在GPU上运行。

在经过一系列处理后，将3个3D的坐标转换为2D的像素输出。


## 顶点输入

现在可以想象由空间中一组顶点坐标的集合构成的一个三角形。

但是注意在OpenGL中只处理（-1,1）区间内的坐标，这范围叫做标准化设备坐标，这个范围以外的坐标不会显示在屏幕上。

所以我们想好一组后可以将其归一化，或者是直接想一组在范围内的坐标，为了更好展示直接把z轴设为0。

```c++
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};
```

接下来就需要用到顶点着色器处理顶点数据，这意味着我们需要把数据传入到GPU中处理，同时还要告诉GPU如何处理这些数据。

使用CPU发送数据到显卡相对较慢，此时就要用到顶点缓冲对象（Vertex Buffer Objects, VBO）来储存传入的数据，
再由GPU进行处理。

```c++
unsigned int VBO;   //标号或者说名字， 又叫缓冲ID，意味着可以有很多个VBO，需要ID区分
glGenBuffers(1, &VBO);  //生成VBO对象
glBindBuffer(GL_ARRAY_BUFFER, VBO);     //绑定缓冲类型，GL_ARRAY_BUFFER表示为顶点缓冲对象，给到VBO上（这里指的是缓冲ID）
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  //将数据给到缓冲GL_ARRAY_BUFFER里
```
glBufferData()函数里第四个参数中指定了显卡如何管理给定的数据：
- GL_STATIC_DRAW ：数据不会或几乎不会改变。
- GL_DYNAMIC_DRAW：数据会被改变很多。
- GL_STREAM_DRAW ：数据每次绘制时都会改变。


### 顶点着色器

接下来我们就需要创建顶点着色器来处理顶点数据，不过在此之前需要了解一下OpenGL中的着色器。

现代OpenGL需要我们至少设置一个顶点和片段着色器，这些着色器是用GLSL语言写的。
~~这表明编写后的着色器无法在VS中编译运行~~

为了编译着色器需要用到GLFW相关函数进行编译
```c++
const char *vertexShaderSource = 
"#version 330 core\n"   //版本
"layout (location = 0) in vec3 aPos;\n" //接收顶点数据，location为位置
"void main()\n"
"{\n"
"   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n" //齐次坐标
"}\0";

unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

### 片段着色器
```c++
const char *fragmentShaderSource = 
"#version 330 core\n"
"out vec4 FragColor;"   //输出值，这里为RGB值
"void main()\n"
"{\n"
"   FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);"  //定义颜色值
"}\0";

unsigned int fragmentShader;
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);
```



## 顶点属性
顶点着色器允许我们指定任何以顶点属性为形式的输入。这使其具有很强的灵活性的同时，它还的确意味着我们必须手动指定输入数据的哪一个部分对应顶点着色器的哪一个顶点属性。所以，我们必须在渲染前指定OpenGL该如何解释顶点数据。

对于之前的三角形顶点数据，应该被解析为下图所示：

![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/vertex_attribute_pointer.png)


```c++
//配置顶点属性
//第一个参数将顶点属性的位置设为0。
//第二个参数指定顶点属性的大小，顶点由3个值组成。
//第三个参数指定数据的类型
//第四个参数表明是否将数据标准化
//第五个参数为步长，每三个值的位置表明一组数据
//最后一个参数表示起始值的偏移量
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);

//以顶点属性位置值为参考，启用顶点属性
glEnableVertexAttribArray(0);
```

## 顶点数组对象
为了不用对每个顶点属性进行配置，于是就有了顶点数组对象(Vertex Array Object, VAO)。
这使在不同顶点数据和属性配置之间切换变得非常简单，只需要绑定不同的VAO就行了。
```c++
unsigned int VAO;
glGenVertexArrays(1, &VAO);
//绑定VAO
glBindVertexArray(VAO);
//把顶点数组复制到缓冲中供OpenGL使用
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
//设置顶点属性指针
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
```

## The triangle
在完成前面一系列操作后，只需要在渲染循环里调用函数即可绘制三角形。
```c++
while (!glfwWindowShouldClose(window))
{
    {
        //input
        

        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        //使用顶点和片段着色器
        glUseProgram(shaderProgram);
        //绑定VAO
        glBindVertexArray(VAO);
        //绘制图形，第二个参数指定了顶点数组的起始索引，第三个参数指定绘制多少顶点
        glDrawArrays(GL_TRIANGLES, 0, 3);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProgram);

    glfwTerminate();
    return 0;
}
```
