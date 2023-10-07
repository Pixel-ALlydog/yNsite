---
title: "5、3D空间(code)"
weight: 6
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

在此之前都只是理论基础，现在可以使用OpenGL构建一个简单的3D场景了。


# 物体
想要一个3D场景，首先需要有个物体，比如正方体。
{{< expand  "Cube" >}}
## 正方体顶点数组
```c++
float Cube[] = {
	-0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
	 0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
	 0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
	 0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
	-0.5f,  0.5f, -0.5f,  0.0f,  0.0f, -1.0f,
	-0.5f, -0.5f, -0.5f,  0.0f,  0.0f, -1.0f,

	-0.5f, -0.5f,  0.5f,  0.0f,  0.0f, 1.0f,
	 0.5f, -0.5f,  0.5f,  0.0f,  0.0f, 1.0f,
	 0.5f,  0.5f,  0.5f,  0.0f,  0.0f, 1.0f,
	 0.5f,  0.5f,  0.5f,  0.0f,  0.0f, 1.0f,
	-0.5f,  0.5f,  0.5f,  0.0f,  0.0f, 1.0f,
	-0.5f, -0.5f,  0.5f,  0.0f,  0.0f, 1.0f,

	-0.5f,  0.5f,  0.5f, -1.0f,  0.0f,  0.0f,
	-0.5f,  0.5f, -0.5f, -1.0f,  0.0f,  0.0f,
	-0.5f, -0.5f, -0.5f, -1.0f,  0.0f,  0.0f,
	-0.5f, -0.5f, -0.5f, -1.0f,  0.0f,  0.0f,
	-0.5f, -0.5f,  0.5f, -1.0f,  0.0f,  0.0f,
	-0.5f,  0.5f,  0.5f, -1.0f,  0.0f,  0.0f,

	 0.5f,  0.5f,  0.5f,  1.0f,  0.0f,  0.0f,
	 0.5f,  0.5f, -0.5f,  1.0f,  0.0f,  0.0f,
	 0.5f, -0.5f, -0.5f,  1.0f,  0.0f,  0.0f,
	 0.5f, -0.5f, -0.5f,  1.0f,  0.0f,  0.0f,
	 0.5f, -0.5f,  0.5f,  1.0f,  0.0f,  0.0f,
	 0.5f,  0.5f,  0.5f,  1.0f,  0.0f,  0.0f,

	-0.5f, -0.5f, -0.5f,  0.0f, -1.0f,  0.0f,
	 0.5f, -0.5f, -0.5f,  0.0f, -1.0f,  0.0f,
	 0.5f, -0.5f,  0.5f,  0.0f, -1.0f,  0.0f,
	 0.5f, -0.5f,  0.5f,  0.0f, -1.0f,  0.0f,
	-0.5f, -0.5f,  0.5f,  0.0f, -1.0f,  0.0f,
	-0.5f, -0.5f, -0.5f,  0.0f, -1.0f,  0.0f,

	-0.5f,  0.5f, -0.5f,  0.0f,  1.0f,  0.0f,
	 0.5f,  0.5f, -0.5f,  0.0f,  1.0f,  0.0f,
	 0.5f,  0.5f,  0.5f,  0.0f,  1.0f,  0.0f,
	 0.5f,  0.5f,  0.5f,  0.0f,  1.0f,  0.0f,
	-0.5f,  0.5f,  0.5f,  0.0f,  1.0f,  0.0f,
	-0.5f,  0.5f, -0.5f,  0.0f,  1.0f,  0.0f
	};
```
{{< /expand >}}

当然这里不只有顶点数据，还有每一面的法线方向，现在还不需要用到，可以在配置顶点属性的时候略过。


# 从头开始吧！

理所应当，我们需要一个窗口：

{{< expand "一个窗口" >}}
```c++
#include<shader.h>
#include<iostream>

#include<glad/glad.h>
#include<GLFW/glfw3.h>

#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>

int main()
{
    glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
	if (window == NULL)
	{
		std::cout << "Failed to create GLFW window" << std::endl;
		glfwTerminate();
		return -1;
	}
	glfwMakeContextCurrent(window);

	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
	{
		std::cout << "Failed to initialize GLAD" << std::endl;
		return -1;
	}

    glViewport(0, 0, 800, 600);
	while (!glfwWindowShouldClose(window)) 
	{
		//输入
		//...

		//渲染
		glClearColor(0.2f, 0.2f, 0.3f, 1.0f);   //改颜色
		glClear(GL_COLOR_BUFFER_BIT);   //清空颜色缓冲，也有其它缓冲如depth buffer、stencil buffer
		
		//检查并调用事件
		glfwSwapBuffers(window);    //交换颜色缓冲，绘制图形（存储window中每个像素颜色值的缓冲）
		glfwPollEvents();    //检查触发时间
	}
	glfwTerminate();
	return 0;
}
```
{{< /expand  >}}


现在加入正方体，并配置顶点属性
```c++
//float cube[] = ...

unsigned int VBO;
glGenBuffers(1, &VBO);
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(cube), cube, GL_STATIC_DRAW);

unsigned int VAO;
glGenVertexArrays(1, &VAO);
glBindVertexArray(VAO);
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(cube), cube, GL_STATIC_DRAW);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
```

有了正方体，接下来就是坐标系变换了。

## MVP和着色器
使用LookAt函数可以省下很多步骤，只需要输入相机位置，一个目标位置和一个表示世界空间中的上向量的向量即可。

perspective函数创建了一个定义了可视空间的大平截头体，任何在这个平截头体以外的东西最后都不会出现在裁剪空间体积内，并且将会受到裁剪。
```c++
Shader ourshader("vshader.vs", "fshader.fs");
//...
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
glm::vec3 cameraUp = glm::vec3(0.0f, 1.0f, 0.0f);

while()
{
	//...
	glm::mat4 model = glm::mat4(1.0f);
	glm::mat4 view = glm::mat4(1.0f);
	glm::mat4 projection;

	view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
	projection = glm::perspective(glm::radians(45.0f), 800.0f / 600.0f, 0.1f, 100.0f);
	//...
}
```

顶点着色器接受顶点数据，还有3个变换矩阵
```glsl
#version 330 core
layout(location = 0) in vec3 aPos;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
	gl_Position = projection * view * model * vec4(aPos, 1.0f);
}
```

片段着色器输出颜色
```glsl
#version 330 core
out vec4 FragColor;
void main()
{
	FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
}
```

最后将MVP给到着色器，在渲染循环中绘制图形
```c++
while()
{
	//...

	//渲染
	//...

	//MVP
	//...

	//启用着色器
	ourshader.use();
	ourshader.setMat4("model", model);
	ourshader.setMat4("view", view);
	ourshader.setMat4("projection", projection);

	glBindVertexArray(VAO);
	glDrawArrays(GL_TRIANGLES, 0, 36);

	//检查并调用事件
	//...
}
```

如果一切没问题，运行后，可以得到这样的图像：

![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/cube.jpg)


## 操控摄像头
这看起来仍然像是2D的样子，是因为摄像机正对着物体。如果能够操控摄像头，看起来就是一个3D的场景了。

### 移动
使用glfwGetKey函数可以捕获对应的操作，再对相机进行平移。

```c++
void processInput(GLFWwindow * window)
{
	float cameraSpeed = 0.05f; // adjust accordingly
    if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
        cameraPos += cameraSpeed * cameraFront;
    if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
        cameraPos -= cameraSpeed * cameraFront;
    if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
        cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
    if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
        cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
}
```
当按下WASD键的任意一个，摄像机的位置都会相应更新。如果我们希望向前或向后移动，我们就把位置向量加上或减去方向向量。如果我们希望向左右移动，我们使用叉乘来创建一个右向量(Right Vector)，并沿着它相应移动就可以了。这样就创建了使用摄像机时熟悉的横移(Strafe)效果。

目前的移动速度是个常量。理论上没什么问题，但是实际情况下根据处理器的能力不同，有些人可能会比其他人每秒绘制更多帧，也就是以更高的频率调用processInput函数。结果就是，根据配置的不同，有些人可能移动很快，而有些人会移动很慢。

图形程序和游戏通常会跟踪一个时间差(Deltatime)变量，它储存了渲染上一帧所用的时间。我们把所有速度都去乘以deltaTime值。结果就是，如果我们的deltaTime很大，就意味着上一帧的渲染花费了更多时间，所以这一帧的速度需要变得更高来平衡渲染所花去的时间。使用这种方法时，无论你的电脑快还是慢，摄像机的速度都会相应平衡，这样每个用户的体验就都一样了。


跟踪两个全局变量来计算出deltaTime值，在每一帧中计算出新的deltaTime以备后用。

```c++
float deltaTime = 0.0f; // 当前帧与上一帧的时间差
float lastFrame = 0.0f; // 上一帧的时间

while()
{
	//...
	float currentFrame = glfwGetTime();
	deltaTime = currentFrame - lastFrame;
	lastFrame = currentFrame;

	processInput(window)
	//...
}
```
有了deltaTime，在计算速度的时候可以将其考虑进去了：
```c++
	float cameraSpeed = 2.5f * deltaTIme;
```


### 旋转

通过移动鼠标获取俯仰角和偏航角，水平的移动影响偏航角，竖直的移动影响俯仰角。它的原理就是，储存上一帧鼠标的位置，在当前帧中我们当前计算鼠标位置与上一帧的位置相差多少。如果水平/竖直差别越大那么俯仰角或偏航角就改变越大，也就是摄像机需要移动更多的距离。

首先要告诉GLFW，它应该隐藏光标，并捕捉(Capture)它。
```c++
glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);
```

定义一个回调函数让GLFW监听鼠标移动事件，xpos和ypos代表当前鼠标的位置。当我们用GLFW注册了回调函数之后，鼠标一移动mouse_callback函数就会被调用：
```c++
void mouse_callback(GLFWwindow* window, double xpos, double ypos);
//...

glfwSetCursorPosCallback(window, mouse_callback);
//...
```

在获取最终方向之前需要：
1. 计算鼠标距上一帧的偏移量。
2. 把偏移量添加到摄像机的俯仰角和偏航角中。
3. 对偏航角和俯仰角进行最大和最小值的限制。
4. 计算方向向量。

第一步是计算鼠标自上一帧的偏移量。我们必须先在程序中储存上一帧的鼠标位置，我们把它的初始值设置为屏幕的中心（屏幕的尺寸是800x600），然后在鼠标的回调函数中我们计算当前帧和上一帧鼠标位置的偏移量：

```c++
float lastX = 400, lastY = 300;

float xoffset = xpos - lastX;
// 注意这里是相反的，因为y坐标是从底部往顶部依次增大的
float yoffset = lastY - ypos; 
lastX = xpos;
lastY = ypos;

//灵敏度
float sensitivity = 0.05f;
xoffset *= sensitivity;
yoffset *= sensitivity;
```

接下来把偏移量加到全局变量pitch和yaw上，同时给摄像机添加一些限制，对于俯仰角，要让用户不能看向高于89度的地方（在90度时视角会发生逆转，所以我们把89度作为极限），同样也不允许小于-89度。这样能够保证用户只能看到天空或脚下，但是不能超越这个限制。我们可以在值超过限制的时候将其改为极限值来实现：
```c++
yaw += xoffset;
pitch += yoffset;

if(pitch > 89.0f)
  pitch =  89.0f;
if(pitch < -89.0f)
  pitch = -89.0f;
```

最后通过俯仰角和偏航角来计算以得到真正的方向向量：
```c++
glm::vec3 front;
front.x = cos(glm::radians(pitch)) * cos(glm::radians(yaw));
front.y = sin(glm::radians(pitch));
front.z = cos(glm::radians(pitch)) * sin(glm::radians(yaw));
cameraFront = glm::normalize(front);
```

如果你现在运行代码，你会发现在窗口第一次获取焦点的时候摄像机会突然跳一下。这个问题产生的原因是，在你的鼠标移动进窗口的那一刻，鼠标回调函数就会被调用，这时候的xpos和ypos会等于鼠标刚刚进入屏幕的那个位置。这通常是一个距离屏幕中心很远的地方，因而产生一个很大的偏移量，所以就会跳了。我们可以简单的使用一个bool变量检验我们是否是第一次获取鼠标输入，如果是，那么我们先把鼠标的初始位置更新为xpos和ypos值，这样就能解决这个问题；接下来的鼠标移动就会使用刚进入的鼠标位置坐标来计算偏移量了：
```c++
if(firstMouse) // 这个bool变量初始时是设定为true的
{
    lastX = xpos;
    lastY = ypos;
    firstMouse = false;
}
```

{{< expand "最终代码" >}}
```c++
float lastX = 400, lastY = 300;
bool firstMouse = true;
float yaw = -90.0f;	
float pitch = 0.0f;

//...

void mouse_callback(GLFWwindow* window, double xpos, double ypos)
{
    if(firstMouse)
    {
        lastX = xpos;
        lastY = ypos;
        firstMouse = false;
    }

    float xoffset = xpos - lastX;
    float yoffset = lastY - ypos; 
    lastX = xpos;
    lastY = ypos;

    float sensitivity = 0.05;
    xoffset *= sensitivity;
    yoffset *= sensitivity;

    yaw   += xoffset;
    pitch += yoffset;

    if(pitch > 89.0f)
        pitch = 89.0f;
    if(pitch < -89.0f)
        pitch = -89.0f;

    glm::vec3 front;
    front.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
    front.y = sin(glm::radians(pitch));
    front.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
    cameraFront = glm::normalize(front);
}
```
{{< /expand >}}



## 丰富场景
一个方块看起来还是比较单调，这里定义一个vec3数组存储方块的位置，之后在循环中多次绘制方块即可
{{< expand "cubposition" >}}
```c++
glm::vec3 position[] = {
		glm::vec3(-6.0f,6.0f,0.0f),
		glm::vec3(-3.0f,6.0f,0.0f),
		glm::vec3(0.0f, 6.0f,0.0f),
		glm::vec3(3.0f, 6.0f,0.0f),
		glm::vec3(6.0f, 6.0f,0.0f),
		glm::vec3(-6.0f,3.0f,0.0f),
		glm::vec3(-3.0f,3.0f,0.0f),
		glm::vec3(0.0f, 3.0f,0.0f),
		glm::vec3(3.0f, 3.0f,0.0f),
		glm::vec3(6.0f, 3.0f,0.0f),
		glm::vec3(-6.0f,0.0f,0.0f),
		glm::vec3(-3.0f,0.0f,0.0f),
		glm::vec3(0.0f, 0.0f,0.0f),
		glm::vec3(3.0f, 0.0f,0.0f),
		glm::vec3(6.0f, 0.0f,0.0f),
		glm::vec3(-6.0f,-3.0f,0.0f),
		glm::vec3(-3.0f,-3.0f,0.0f),
		glm::vec3(0.0f, -3.0f,0.0f),
		glm::vec3(3.0f, -3.0f,0.0f),
		glm::vec3(6.0f, -3.0f,0.0f),
		glm::vec3(-6.0f,-6.0f,0.0f),
		glm::vec3(-3.0f,-6.0f,0.0f),
		glm::vec3(0.0f, -6.0f,0.0f),
		glm::vec3(3.0f, -6.0f,0.0f),
		glm::vec3(6.0f, -6.0f,0.0f)
	};
```
{{< /expand >}}


同时将每个方块的位置作为颜色值进行绘制，在片段着色器中定义一个uinform接受位置数据。由于OpenGL的RGB值在[0,1]之间，所以对位置的值进行归一化。使用glfwGetTime()可以获得运行的秒数。
```c++
	while()
	{
		//...

		glBindVertexArray(VAO);
		for (unsigned int i = 0; i < 25; i++)
		{
			glm::mat4 model = glm::mat4(1.0f);;
			model = glm::translate(model, position[i]);
			model = glm::rotate(model, glm::radians(5*time), glm::vec3(1.0f, 0.3f, 0.5f));

			ourshader.setVec3("color", (position[i]-position[20])/(position[4]-position[20]));
			ourshader.setMat4("model", model);

			glDrawArrays(GL_TRIANGLES, 0, 36);
		}

		//...
	}
```

看起来像这样：
![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/cube_more.png)