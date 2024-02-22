---
title: "8、深度测试"
weight: 8
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---
# 深度缓冲(z-buffer)
深度缓冲与颜色缓冲一样，在每个片段中存储了信息。深度缓冲是由窗口系统自动创建的，它会以16、24或32位float的形式储存它的深度值。在大部分的系统中，深度缓冲的精度都是24位的。

# 深度测试(Depth testing)
在opengl中启用时，会将一个片段的深度值与深度缓冲进行对比，然后将缓冲更新为新的深度值（离屏幕或相机近的留下）。

深度缓冲是在片段着色器运行之后，在屏幕空间坐标中运行的（不管片段的前后，遍历所有片段覆盖的像素，比对深度值存储深度值到缓冲里）。

默认情况禁用，使用glEnable(GL_DEPTH_TEST)启用

使用[glDepthFunc](https://docs.gl/gl3/glDepthFunc)指定用于深度缓冲区的比较方法

## 深度值精度
深度缓冲包含了一个介于0.0和1.0之间的深度值，对于z轴的距离使用线性变换进行归一化：
{{< katex display >}}
F_{depth} = \frac{z - near}{far - near}
{{< /katex >}}

实践中使用非线性的深度方程，可以使z在很小的时候有更高的精度：
{{< katex display >}}
F_{depth} = \frac{1/z - 1/near}{1/far - 1/near}
{{< /katex >}}


## 深度冲突
在物体片段重叠或非常近时，无法正确显示结果。
为了避免深度冲突可以使用几个方法：
- 不要把物体摆的太靠近
- 可以将近平面设置远一点
- 使用更高精度的深度缓冲，但这会牺牲一些性能
