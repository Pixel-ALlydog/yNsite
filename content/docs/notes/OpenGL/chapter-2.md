---
title: "2、着色器，纹理"
weight: 3
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# 着色器

为了更方便编写着色器，可以使用C++文件流读取着色器内容，参考[这个链接](https://learnopengl.com/code_viewer_gh.php?code=includes/learnopengl/shader_m.h)。


## GLSL
着色器是使用一种叫GLSL的类C语言写成的。GLSL是为图形计算量身定制的，它包含一些针对向量和矩阵操作的有用特性。

着色器的开头总是要声明版本，接着是输入和输出变量、uniform和main函数。每个着色器的入口点都是main函数，在这个函数中我们处理所有的输入变量，并将结果输出到输出变量中。

```glsl
#version version_number
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

int main()
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```

### 数据类型
GLSL中的向量是一个可以包含有2、3或者4个分量的容器，分量的类型可以是前面默认基础类型的任意一个。
|  类型      | 	   描述                  |
|--------|------------------------------| 
| vecn   | 包含n个float分量的默认向量     |
| bvecn  | 包含n个bool分量的向量          |
| ivecn  | 包含n个int分量的向量           |
| uvecn  | 包含n个unsigned int分量的向量  |
| dvecn  | 包含n个double分量的向量        |

向量这一数据类型也允许一些有趣而灵活的分量选择方式，叫做重组(Swizzling)。
可以使用上面4个字母(x,y,z,w)任意组合来创建一个和原来向量一样长的（同类型）新向量，只要原来向量有那些分量即可；
```glsl
vec3 dir = vec3(1,2,3);
vec4 rgba = dir.zyxx;
```

### 输入与输出
GLSL定义了in和out关键字在各个着色器间传递数据。

顶点着色器应该接收的是一种特殊形式的输入，否则就会效率低下。顶点着色器的输入特殊在，它从顶点数据中直接接收输入。

片段着色器需要一个vec4颜色输出变量，因为片段着色器需要生成一个最终输出的颜色。如果你在片段着色器没有定义输出颜色，OpenGL会把你的物体渲染为黑色（或白色）。

## Uniform
Uniform是一种从CPU中的应用向GPU中的着色器发送数据的方式，这与顶点属性稍有不同。
- uniform是全局的(Global)，在每个着色器对象中是独一无二的，可以被着色器程序的任意着色器在任意阶段访问。
- 无论把uniform值设置成什么，uniform会一直保存它们的数据，直到它们被重置或更新。

```glsl
#version 330 core
out vec4 FragColor;
uniform vec4 Color;
void main()
{
    FragColor = Color;
}
```

```c++
float timeValue = glfwGetTime();
float greenValue = (sin(timeValue) / 2.0f) + 0.5f;

//找地址（位置值）
int vertexColorLocation = glGetUniformLocation(shaderProgram, "Color");
//激活着色器
glUseProgram(shaderProgram);
//将greenValue给到Color的地址上
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

## 顶点属性（more）
在之前定义的三角形数组中只给定了三个顶点的坐标值，在这个数组里可以添加更多的属性，如颜色值、法线等等。
然后配置顶点属性存储在VAO中。
```c++
float vertices[] = {
    // 位置              // 颜色
     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 顶部
};
```
在顶点着色其中要接受两种属性，再将颜色值给到片段着色器
```glsl
#version 330 core
layout(location = 0) in vec3 aPos;
layout(location = 1) in vec3 aColor;

out vec3 Color;
void main(){
    gl_Position = vec4(aPos,1.0);
    Color = aColor;
}
```

此时在片段着色器中，就不用uniform传递颜色了
```glsl
#version 330 core
out vec4 FragColor;  
in vec3 Color;

void main()
{
    FragColor = vec4(Color, 1.0);
}
```

有了更多的顶点属性，就必须重新配置属性指针。
```c++
// 位置属性
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
// 颜色属性
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3* sizeof(float)));
glEnableVertexAttribArray(1);
```

# 纹理
在之前的三角形中，我们可以给到不同的颜色，但似乎在细节上仍然不够。
如果想让图形看起来更加真实，可以给定更多顶点，赋予更多颜色，当然这会使得电脑的开销变得更大。

还有一种方法就是给模型贴上纹理，这可以大大丰富物体细节，同时节省了开销。

为了能够把纹理映射(Map)到三角形上，我们需要指定三角形的每个顶点各自对应纹理的哪个部分。这样每个顶点就会关联着一个纹理坐标(Texture Coordinate)，用来标明该从纹理图像的哪个部分采样（采集片段颜色）。之后在图形的其它片段上进行片段插值(Fragment Interpolation)。

纹理坐标在x和y轴上，范围为0到1之间（注意使用的是2D纹理图像）。使用纹理坐标获取纹理颜色叫做采样(Sampling)。纹理坐标起始于(0, 0)，也就是纹理图片的左下角，终始于(1, 1)，即纹理图片的右上角。

![](https://cdn.jsdelivr.net/gh/Pixel-ALlydog/blog-img/opengl/tex_coords.png)

只需要给顶点着色器传递这三个纹理坐标就足够了，接下来它们会被传片段着色器中，它会为每个片段进行纹理坐标的插值。

看起来就像这样：
```c++
float texCoords[] = {
    0.0f, 0.0f, // 左下角
    1.0f, 0.0f, // 右下角
    0.5f, 1.0f // 上中
};
```

## 纹理环绕方式
对于超出纹理范围的情况，OpenGL默认重复这个纹理图像，也提供了一些别的选择：

|    方法   | 	       描述                     |
|--------|------------------------------| 
| GL_REPEAT   | 对纹理的默认行为。重复纹理图像。     |
| GL_MIRRORED_REPEAT  | 和GL_REPEAT一样，但每次重复图片是镜像放置的。    |
| GL_CLAMP_TO_EDGE  | 纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。  |
| GL_CLAMP_TO_BORDER  | 超出的坐标为用户指定的边缘颜色。  |


## 纹理过滤
纹理坐标不依赖于分辨率(Resolution)，它可以是任意浮点值，所以OpenGL需要知道怎样将纹理像素(Texture Pixel)映射到纹理坐标。对于一些比较大的物体，可纹理分辨率较低的时候这就比较重要。比较常用的两种方式分别为邻近过滤和线性过滤。~~（感觉和数字图像处理的插值方法有点像）~~

- GL_NEAREST（邻近过滤，Nearest Neighbor Filtering）是OpenGL默认的纹理过滤方式。当设置为GL_NEAREST的时候，OpenGL会选择中心点最接近纹理坐标的那个像素。
- GL_LINER（线性过滤，linear Filtering）它会基于纹理坐标附近的纹理像素，计算出一个插值，近似出这些纹理像素之间的颜色。一个纹理像素的中心距离纹理坐标越近，那么这个纹理像素的颜色对最终的样本颜色的贡献越大。


## 多级渐远纹理
 当一个纹理映射在一个物体上，我们比较容易发现，物体距离近时纹理图案清晰（假设纹理分辨率高），距离远时会出现一些失真。这是因为屏幕中每个像素的采样点是有限的，物体距离近时，有很多个采样点对其进行采样，表现出来就会很清晰。但是一旦物体距离远到一定程度时，屏幕像素中能对其采样的点就会减少，并且纹理分率依然很高，此时就会出现失真现象比如产生摩尔纹。

 OpenGL使用一种叫做多级渐远纹理(Mipmap)的概念来解决这个问题，简单来说就是一系列的纹理图像，后一个纹理图像是前一个的二分之一。多级渐远纹理背后的理念很简单：距观察者的距离超过一定的阈值，OpenGL会使用不同的多级渐远纹理，即最适合物体的距离的那个。同时，多级渐远纹理另一加分之处是它的性能非常好。

 OpenGL有一个glGenerateMipmaps函数，在创建完一个纹理后调用它OpenGL就会承担接下来的所有工作了。

 在渲染中切换多级渐远纹理级别(Level)时，OpenGL在两个不同级别的多级渐远纹理层之间会产生不真实的生硬边界。就像普通的纹理过滤一样，切换多级渐远纹理级别时也可以在两个不同多级渐远纹理级别之间使用NEAREST和LINEAR过滤。为了指定不同多级渐远纹理级别之间的过滤方式。


 |   算法     | 	     描述                      |
|--------|------------------------------| 
| GL_NEAREST_MIPMAP_NEAREST   | 使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样     |
| GL_LINEAR_MIPMAP_NEAREST  | 使用最邻近的多级渐远纹理级别，并使用线性插值进行采样    |
| GL_NEAREST_MIPMAP_LINEAR | 在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样  |
| GL_LINEAR_MIPMAP_LINEAR  | 在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样  |


## 生成纹理

```c++
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
// 为当前绑定的纹理对象设置环绕
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);   
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
//设置过滤方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
// 加载并生成纹理
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
if (data)
{
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    glGenerateMipmap(GL_TEXTURE_2D);
}
else
{
    std::cout << "Failed to load texture" << std::endl;
}
stbi_image_free(data);
```

关于glTexImage2D
- 第一个参数指定了纹理目标(Target)。设置为GL_TEXTURE_2D意味着会生成与当前绑定的纹理对象在同一个目标上的纹理（任何绑定到GL_TEXTURE_1D和GL_TEXTURE_3D的纹理不会受到影响）。
- 第二个参数为纹理指定多级渐远纹理的级别，如果你希望单独手动设置每个多级渐远纹理的级别的话。这里我们填0，也就是基本级别。
- 第三个参数告诉OpenGL我们希望把纹理储存为何种格式。我们的图像只有RGB值，因此我们也把纹理储存为RGB值。
- 第四个和第五个参数设置最终的纹理的宽度和高度。我们之前加载图像的时候储存了它们，所以我们使用对应的变量。
下个参数应该总是被设为0（历史遗留的问题）。
- 第七第八个参数定义了源图的格式和数据类型。我们使用RGB值加载这个图像，并把它们储存为char(byte)数组，我们将会传入对应值。
- 最后一个参数是真正的图像数据。

当调用glTexImage2D时，当前绑定的纹理对象就会被附加上纹理图像。然而，目前只有基本级别(Base-level)的纹理图像被加载了，如果要使用多级渐远纹理，我们必须手动设置所有不同的图像（不断递增第二个参数）。或者，直接在生成纹理之后调用glGenerateMipmap。这会为当前绑定的纹理自动生成所有需要的多级渐远纹理。


## 应用纹理
此时需要告诉OpenGl如何采样（显示到屏幕上）纹理，在长方形顶点属性里添加纹理坐标属性。
```c++
float vertices[] = {
//     ---- 位置 ----       ---- 颜色 ----     - 纹理坐标 -
     0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f,   // 右上
     0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f,   // 左下
    -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f    // 左上
};
```

```c++
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
glEnableVertexAttribArray(2);
```

在顶点着色器中，将纹理坐标传给片段着色器：
```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aTexCoord;

out vec3 ourColor;
out vec2 TexCoord;

void main()
{
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor;
    TexCoord = aTexCoord;
}
```

此时有了纹理坐标，为了渲染纹理，我们还需要将纹理给到片段着色器（好让片段着色器采样并输出）。

GLSL有一个供纹理对象使用的内建数据类型，叫做采样器(Sampler)，它以纹理类型作为后缀，比如sampler1D、sampler3D，或在我们的例子中的sampler2D。可以简单声明一个uniform sampler2D把一个纹理添加到片段着色器中。

```glsl
#version 330 core
out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

uniform sampler2D ourTexture;

void main()
{
    FragColor = texture(ourTexture, TexCoord);
}
```
使用GLSL内建的texture函数来采样纹理的颜色，它第一个参数是纹理采样器，第二个参数是对应的纹理坐标。texture函数会使用之前设置的纹理参数对相应的颜色值进行采样。这个片段着色器的输出就是纹理的（插值）纹理坐标上的(过滤后的)颜色。

最后在渲染循环中调用glDrawElements之前绑定纹理了，它会自动把纹理赋值给片段着色器的采样器。

```c++
while (!glfwWindowShouldClose(window))
    {
        // input

        // render
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // bind Texture
        glBindTexture(GL_TEXTURE_2D, texture);

        // render container
        ourShader.use();
        glBindVertexArray(VAO);
        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }
```

## 纹理单元
sampler2D变量是个uniform，我们却不用glUniform给它赋值。使用glUniform1i，我们可以给纹理采样器分配一个位置值，这样的话我们能够在一个片段着色器中设置多个纹理。

一个纹理的位置值通常称为一个纹理单元(Texture Unit)。一个纹理的默认纹理单元是0，它是默认的激活纹理单元。这可以让我们一次绑定多个纹理，只要首先激活对应的纹理单元。

OpenGL至少保证有16个纹理单元供你使用，也就是说你可以激活从GL_TEXTURE0到GL_TEXTRUE15。它们都是按顺序定义的，所以我们也可以通过GL_TEXTURE0 + 8的方式获得GL_TEXTURE8，这在当我们需要循环一些纹理单元的时候会很有用。

```glsl
#version 330 core
...

uniform sampler2D texture1;
uniform sampler2D texture2;

void main()
{
    FragColor = mix(texture(texture1, TexCoord), texture(texture2, TexCoord), 0.2);
}
```

```c++
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture1);
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);

glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```
还要通过使用glUniform1i设置每个采样器的方式告诉OpenGL每个着色器采样器属于哪个纹理单元。我们只需要设置一次即可，所以这个会放在渲染循环的前面：
```c++
ourShader.use(); // 不要忘记在设置uniform变量之前激活着色器程序！
glUniform1i(glGetUniformLocation(ourShader.ID, "texture1"), 0); // 手动设置
ourShader.setInt("texture2", 1); // 或者使用着色器类设置

while(...) 
{
    [...]
}
```

这样就可以显示多个纹理了。