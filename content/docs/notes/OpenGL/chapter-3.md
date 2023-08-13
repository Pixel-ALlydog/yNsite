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
# 变换

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

### 与向量
