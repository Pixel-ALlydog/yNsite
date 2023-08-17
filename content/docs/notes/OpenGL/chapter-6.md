---
title: "6、冯氏光照"
weight: 7
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# 概述

这是一个简单的光照模型，是对现实情况的近似。

冯氏光照模型的主要结构由3个分量组成：环境(Ambient)、漫反射(Diffuse)和镜面(Specular)光照。

![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/basic_lighting_phong.png)

- 环境光照(Ambient Lighting)：即使在黑暗的情况下，世界上通常也仍然有一些光亮（月亮、远处的光），所以物体几乎永远不会是完全黑暗的。为了模拟这个，会使用一个环境光照常量，它永远会给物体一些颜色。
- 漫反射光照(Diffuse Lighting)：模拟光源对物体的方向性影响(Directional Impact)。它是冯氏光照模型中视觉上最显著的分量。物体的某一部分越是正对着光源，它就会越亮。
- 镜面光照(Specular Lighting)：模拟有光泽物体上面出现的亮点。镜面光照的颜色相比于物体的颜色会更倾向于光的颜色。


## 环境光照

通常一个物体被照亮，受到的光不只有直接光照，也有经过很多次反射的间接光照。光可以在各个物体表面上反射，对物体产生影响。
考虑到这种情况的算法叫做全局照明(Global Illumination)算法，但是这种算法既开销高昂又极其复杂。

所以我们将会先使用一个简化的全局照明模型，即环境光照。

只需要使用一个很小的常量（光照）颜色，添加到物体片段的最终颜色中，这样子的话即便场景中没有直接的光源也能看起来存在有一些发散的光。

## 漫反射光照

一束光照在物体表面上有部分光会向周围发散，一般认为这种发散是均匀的，也就是从各处看颜色应该是一样的。

同时因为接触面与光线之间存在角度，使得不同角度下的接触面反射的光，颜色是不一样的，有些面会亮一些，有些则会暗一些。

这个角度是该接触面的法线与光线的夹角。

![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/diffuse_light.png)

此时假设光照强度为{{< katex >}}E{{< /katex >}}，与接触面法线的夹角为{{< katex >}}\theta{{< /katex >}}，那么就可以知道慢反射的强度为：
{{< katex display >}}
L_d = k_d \cdot E \cdot cos\theta
{{< /katex >}}

已经很接近漫反射的结果了，现在暂时还不考虑光线的衰减。

那么考虑到反射不为负数的情况，并且知道夹角是由入射方向({{< katex >}}L{{< /katex >}})点成接触面的法相向量({{< katex >}}N{{< /katex >}})得到，公式可以改为：
{{< katex display >}}
L_d = k_d \cdot E \cdot max(0, L \cdot N)
{{< /katex >}}

只需要在0与{{< katex >}}cos\theta{{< /katex >}}之间取最大值就好，这样当{{< katex >}}cos\theta{{< /katex >}}为负数时就直接取0。


## 镜面光照

还有部分光线会与接触面直接反射，这些反射光线与摄像机的观察方向有关。反射光线的方向是已知的，只需要计算反射向量与观察方向的角度差，它们之间夹角越小，镜面光的作用就越大，也就是物体表面的高光亮度。

![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/basic_lighting_specular_theory.png)

设摄像机的方向向量为{{< katex >}}V{{< /katex >}}

公式表示为：
{{< katex display >}}
L_s = k_s \cdot E \cdot max(0, R \cdot V)^p
{{< /katex >}}

p次幂可以近似物体反射光的能力，即反光度(Shininess)。一个物体的反光度越高，反射光的能力越强，散射得越少，高光点就会越小。


# 实现
首先定义一个光源
```c++
glm::vec3 lightcolor = glm::vec3(1.0f,1.0f,1.0f);
glm::vec3 lightpos = glm::vec3(7.0f,7.0f,5.0f);
```

重新配置顶点属性，得到法线向量
```c++
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3 * sizeof(float)));
glEnableVertexAttribArray(1);
```

再将光源和相机位置传入着色器
```c++
ourshader.setVec3("lightcolor", lightcolor);
ourshader.setVec3("lightpos", lightpos);
ourshader.setVec3("camerapos", cameraPos);
```

在顶点着色器中，接受法线向量并将法线给到片段着色器，同时计算片段位置，我们会在世界空间中进行所有的光照计算，因此我们需要一个在世界空间中的顶点位置。我们可以通过把顶点位置属性乘以模型矩阵（不是观察和投影矩阵）来把它变换到世界空间坐标。
```glsl
//...
layout(location = 1) in vec3 aNormal;

//...
out vec3 Normal;
out vec3 FragPos;

void main()
{
    //...
    Normal = aNormal;
    FragPos = vec3(model*vec4(aPos,1.0));
}
```

在片段着色器中，接受数据后，计算光线入射向量，反射向量，观察向量，接下来就可以完成冯氏光照了
```glsl
//...
in vec3 FragPos;
in vec3 Normal;

uniform vec3 lightcolor;
uniform vec3 lightpos;
uniform vec3 camerapos;

void main()
{
    vec3 normal = normalize(Normal);
    vec3 lightdir = normalize(lightpos - FragPos);
    vec3 lightref = reflect(-lightdir, normal);
    vec3 viewdir = normalize(camerapos, FragPos);

    float ka = 0.1f
    vec3 ambient = ka * lightcolor;

    float diffuse = max(0.0,dot(lightDir,normal));
	vec3 diff = diffuse * lightcolor;
	
	float specular = pow(max(0.0,dot(lightRef, viewdir)), 32);
	vec3 spec = specular * lightcolor;

	vec3 result = (ambient + diff + spec) * color;
	FragColor = vec4(result,1.0f);
}
```

运行后应可以看到:

![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/phong_lighting.png)

这看起来还稍微有点问题，后面的片段也跟着显示出来了。

之所以这样是因为OpenGL是一个三角形一个三角形地来绘制你的立方体的，所以即便之前那里有东西它也会覆盖之前的像素。因为这个原因，有些三角形会被绘制在其它三角形上面，虽然它们本不应该是被覆盖的。


## Z-buffer

OpenGL存储它的所有深度信息于一个Z缓冲(Z-buffer)中，也被称为深度缓冲(Depth Buffer)。GLFW会自动为你生成这样一个缓冲（就像它也有一个颜色缓冲来存储输出图像的颜色）。深度值存储在每个片段里面（作为片段的z值），当片段想要输出它的颜色时，OpenGL会将它的深度值和z缓冲进行比较，如果当前的片段在其它片段之后，它将会被丢弃，否则将会覆盖。这个过程称为深度测试(Depth Testing)，它是由OpenGL自动完成的。

如果我们想要确定OpenGL真的执行了深度测试，首先我们要告诉OpenGL我们想要启用深度测试；它默认是关闭的。我们可以通过glEnable函数来开启深度测试。glEnable和glDisable函数允许我们启用或禁用某个OpenGL功能。这个功能会一直保持启用/禁用状态，直到另一个调用来禁用/启用它。现在我们想启用深度测试，需要开启GL_DEPTH_TEST：
```c++
glEnable(GL_DEPTH_TEST);
```
因为我们使用了深度测试，我们也想要在每次渲染迭代之前清除深度缓冲（否则前一帧的深度信息仍然保存在缓冲中）。就像清除颜色缓冲一样，我们可以通过在glClear函数中指定DEPTH_BUFFER_BIT位来清除深度缓冲：
```c++
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

启用深度测试后，看起来应该是这样：
![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/phong_lighting_depth.png)