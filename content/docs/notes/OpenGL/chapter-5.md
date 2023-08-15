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
	glm::mat4 model = mat4(1.0f);
	glm::mat4 view = mat4(1.0f);
	glm::mat4 projection;

	view = glm::lookAt(cmaeraPos, cameraPos + cameraFront, cameraUp);
	projection = glm::(glm::radians(45.0f), 800.0f / 600.0f, 0.1f, 100.0f);
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
	ourshader.use()
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

