---
title: "3、变换"
weight: 4
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---
# 物体运动

想要让物体运动，就要用到矩阵(Matrix)对物体进行变换(Transform)。不过在此之前需要了解相关的数学运算。

# 向量

一个有大小有方向的量，一般这样表示
{{< katex display >}}
\vec v = 
\left(\begin{matrix}
   x \\
   y \\
   z 
  \end{matrix}\right)
{{< /katex >}}

## 向量运算

### 与标量

{{< katex display >}}
\vec v + u = 
\left(\begin{matrix}
   x + u\\
   y + u\\
   z + u
  \end{matrix}\right)
{{< /katex >}}

同时也可以做减法、乘除运算，但减法和除法只能向量在前面。

### 与向量

假设两个向量{{< katex >}}\vec u, \vec v{{< /katex >}}


{{< katex display >}}
\vec u = 
\left(\begin{matrix}
   x_1 \\
   y_1 \\
   z_1 
  \end{matrix}\right)
, 
\vec v = 
\left(\begin{matrix}
   x_2 \\
   y_2 \\
   z_2 
  \end{matrix}\right)
{{< /katex >}}

加减运算
{{< katex display >}}
\vec u \pm \vec v = 
\left(\begin{matrix}
   x_1 \pm x_2\\
   y_1 \pm y_2\\
   z_1 \pm z_2
  \end{matrix}\right)
{{< /katex >}}

### 点乘
点乘结果为一个标量
{{< katex display >}}
\vec u \cdot \vec v = ||\vec u|| \cdot ||\vec v|| \cdot cos\theta
{{< /katex >}}
也可以写成这样
{{< katex display >}}
\vec u \cdot \vec v = x_1 \cdot x_2 + y_1\cdot y_2 + z_1\cdot z_2
{{< /katex >}}


### 叉乘
叉乘只在3D空间中有定义，这需要两个不平行向量作为输入，结果为一个向量，与两输入向量正交。

{{< katex display >}}
\vec u \times \vec v = 
\left(\begin{matrix}
   y_1 \cdot z_2 - z_1 \cdot y_2\\
   z_1 \cdot x_2 - x_1 \cdot z_2\\
   x_1 \cdot y_2 - y_1 \cdot x_2
  \end{matrix}\right)
{{< /katex >}}


# 矩阵
简单来说矩阵就是一个矩形的数字、符号或表达式数组。矩阵中每一项叫做矩阵的元素(Element)。

一般用大写字母表示
{{< katex display >}}
T= \left[\begin{matrix}
   a & b & c  & p\\
   d & e & f  & q \\
   g & h & i  & r \\
   l & m & n  & s
  \end{matrix}\right]
{{< /katex >}}

## 矩阵运算

### 与标量
加减运算
{{< katex display >}}
\left[\begin{matrix}
   a & b\\
   c & d \\
  \end{matrix}\right]
\pm x = 
\left[\begin{matrix}
   a \pm x & b \pm x\\
   c \pm x & d \pm x \\
  \end{matrix}\right]
{{< /katex >}}
数学上是没有矩阵与标量相加减的运算的，但是很多线性代数的库都对它有支持（比如GLM）。

乘除运算
{{< katex display >}}
\left[\begin{matrix}
   a & b\\
   c & d \\
  \end{matrix}\right]
\cdot x = 
\left[\begin{matrix}
   a \cdot x & b \cdot x\\
   c \cdot x & d \cdot x \\
  \end{matrix}\right]
{{< /katex >}}
满足交换律，除法与乘法类似

### 与矩阵
加减运算
{{< katex display >}}
\left[\begin{matrix}
   a & b\\
   c & d \\
  \end{matrix}\right]
\pm 
\left[\begin{matrix}
   e & f\\
   g & h \\
\end{matrix}\right] = 
\left[\begin{matrix}
   a \pm e & b \pm f\\
   c \pm g & d \pm h \\
  \end{matrix}\right]
{{< /katex >}}

**矩阵相乘**

两个矩阵相乘时需要满足一些条件
- 只有当左侧矩阵的列数与右侧矩阵的行数相等，两个矩阵才能相乘。
- 矩阵相乘不遵守交换律(Commutative)，也就是说A⋅B≠B⋅A。
{{< katex display >}}
\left[\begin{matrix}
   a & b & c\\
   d & e & f\\
   g & h & i
\end{matrix}\right]
\cdot
\left[\begin{matrix}
   x_1 & x_2 & x_3\\
   y_1 & y_2 & y_3\\
   z_1 & z_2 & z_3
  \end{matrix}\right] = 
\left[\begin{matrix}
   ax_1+by_1+cz_1 & ax_2 + by_2 + cz_3 & ax_3 + by_3 + cz_3\\
   dx_1+ey_1+fz_1 & dx_2 + ey_2 + fz_3 & dx_3 + ey_3 + fz_3\\
   gx_1+hy_1+iz_1 & gx_2 + hy_2 + iz_3 & gx_3 + hy_3 + iz_3
  \end{matrix}\right]
{{< /katex >}}

# 变换
在OpenGL中通常使用4*4的变换矩阵（这会使得计算更加方便）。

假设一组顶点
{{< katex display >}}
A = 
\left[\begin{matrix}
   x_1 & x_2 & x_3 \\
   y_1 & y_2 & y_3 \\
   z_1 & z_2 & z_3 \\
   1   & 1   & 1
  \end{matrix}\right]
{{< /katex >}}


这时就可以通过数学运算对顶点进行各种操作。比如缩放（Scaling）
{{< katex display >}}
n \cdot A = 
\left[\begin{matrix}
   nx_1 & nx_2 & nx_3 \\
   ny_1 & ny_2 & ny_3 \\
   nz_1 & nz_2 & nz_3 \\
   n    & n    & n
  \end{matrix}\right]
{{< /katex >}}


但这样并不完全对，我们并不想将最后一行也变为n，这会就要用到矩阵相乘，现在再设一组矩阵，叫做变换矩阵
{{< katex display >}}
T = 
\left[\begin{matrix}
   a & 0 & 0 & 0\\
   0 & b & 0 & 0\\
   0 & 0 & c & 0\\
   0 & 0 & 0 & 1
  \end{matrix}\right]
{{< /katex >}}


此时再做乘法，就可以在各个轴上进行缩放。
为了方便这里只用一个顶点运算，之后也只使用一个顶点
{{< katex display >}}
\left[\begin{matrix}
   a & 0 & 0 & 0 \\
   0 & b & 0 & 0 \\
   0 & 0 & c & 0 \\
   0 & 0 & 0 & 1
\end{matrix}\right]
\cdot
\left[\begin{matrix}
   x\\
   y\\
   z\\
   1
  \end{matrix}\right] = 
\left[\begin{matrix}
   ax\\
   by\\
   cz\\
   1
  \end{matrix}\right]
{{< /katex >}}

## 变换矩阵
顶点数据总是不变的，我们只需要改变变换矩阵就可对物体进行各种操作。现在设变换后的顶点数组为
{{< katex  >}}P{{< /katex  >}}
可以知道
{{< katex display >}}
P = T\cdot A
{{< /katex  >}}

### 平移(Translation)
若想进行平移操作，则变换矩阵为
{{< katex display >}}
T = 
\left[\begin{matrix}
   1 & 0 & 0 & t_x \\
   0 & 1 & 0 & t_y \\
   0 & 0 & 1 & t_z \\
   0 & 0 & 0 & 1
  \end{matrix}\right]
{{< /katex  >}}

变换后的顶点数据为
{{< katex display >}}
P = 
\left[\begin{matrix}
   1 & 0 & 0 & t_x \\
   0 & 1 & 0 & t_y \\
   0 & 0 & 1 & t_z \\
   0 & 0 & 0 & 1
  \end{matrix}\right]
\cdot 
\left[\begin{matrix}
   x\\
   y\\
   z\\
   1
  \end{matrix}\right] =
  \left[\begin{matrix}
   x + t_x\\
   y + t_y\\
   z + t_z\\
   1
  \end{matrix}\right]
{{< /katex  >}}

### 旋转(Rotation)
旋转则比较复杂，在3D空间中旋转需要定义一个角和一个旋转轴(Rotation Axis)。
物体会沿着给定的旋转轴旋转特定角度。

沿x轴旋转：
{{< katex display >}}
T = 
\left[\begin{matrix}
   1 & 0 & 0 & 0 \\
   0 & cos\theta & -sin\theta & 0 \\
   0 & sin\theta & cos\theta & 0 \\
   0 & 0 & 0 & 1
  \end{matrix}\right]
{{< /katex  >}}

沿y轴旋转：
{{< katex display >}}
T = 
\left[\begin{matrix}
   cos\theta & 0 & sin\theta & 0 \\
   0 & 1 & 0 & 0 \\
   -sin\theta & 0 & cos\theta & 0 \\
   0 & 0 & 0 & 1
  \end{matrix}\right]
{{< /katex  >}}

沿z轴旋转：
{{< katex display >}}
T = 
\left[\begin{matrix}
   cos\theta & -sin\theta & 0 & 0 \\
   sin\theta & cos\theta & 0 & 0 \\
   0 & 0 & 1 & 0 \\
   0 & 0 & 0 & 1
  \end{matrix}\right]
{{< /katex  >}}

虽然这样已经可以完成旋转操作，但是仍有一个问题，这会导致万向锁（Gimbal Lock）~~旋转轴重合~~，可能也许会在以后其它地方讲到，现在只需要知道有这么一个问题就行。

## 变换顺序
现在可以假设各种变换矩阵，R（旋转）、S（缩放）、T（平移），若想先平移后再旋转再进缩放，则可以这样表示
{{< katex display >}}
P = S\cdot R\cdot T\cdot A
{{< /katex  >}}


# 回到OpenGL
现在有了变换矩阵后，就可以开始各种操作了。

## GLM
GLM是OpenGL Mathematics的缩写，它是一个只有头文件的库，也就是说我们只需包含对应的头文件就行了，不用链接和编译。GLM可以在它们的[网站](https://glm.g-truc.net/0.9.9/index.html)上下载。

```c++
glm::mat4 trans = glm::mat4(1.0f);
//平移
trans = glm::translate(trans, glm::vec3(1.0f, 1.0f, 0.0));
//旋转
trans = glm::rotate(trans, glm::radians(90.0f), glm::vec3(0,0,1.0));
//缩放
trans = glm::scale(trans,glm::vec3(0.5,0.5,0.5));
//传给着色器
unsigned int transformLoc = glGetUniformLocation(ourShader.ID, "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
```
首先查询uniform变量的地址，然后用有Matrix4fv后缀的glUniform函数把矩阵数据发送给着色器。

第一个参数是uniform的位置值。
第二个参数告诉OpenGL我们将要发送多少个矩阵，这里是1。
第三个参数询问我们是否希望对我们的矩阵进行转置(Transpose)，也就是说交换我们矩阵的行和列。

OpenGL开发者通常使用一种内部矩阵布局，叫做列主序(Column-major Ordering)布局。GLM的默认布局就是列主序，所以并不需要转置矩阵，我们填GL_FALSE。最后一个参数是真正的矩阵数据，但是GLM并不是把它们的矩阵储存为OpenGL所希望接受的那种，因此我们要先用GLM的自带的函数value_ptr来变换这些数据。


接下来就是将变换矩阵传给顶点着色器，定义一个uniform的mat4变量，接受数据
```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;

out vec2 TexCoord;

uniform mat4 transform;

void main()
{
    gl_Position = transform * vec4(aPos, 1.0f);
    TexCoord = vec2(aTexCoord.x, 1.0 - aTexCoord.y);
}
```

