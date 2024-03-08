---
title: "11、帧缓冲"
weight: 12
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# 帧缓冲
OpenGL允许我们定义自己的帧缓冲，也就是我们能确定自己的颜色缓存，深度缓存和模板缓冲。

一个完整的帧缓冲需要满足：
- 附加至少一个缓冲（颜色、深度、模板）
- 至少由一个颜色附件
- 所有的附件都必须是完整的
- 每个缓冲都应该有相同的样本数

在创建并绑定了帧缓冲后，需要为帧缓冲创建一些附件，并附加到帧缓冲上。

## 纹理附件
当把一个纹理附加到帧缓冲的时候，所有的渲染指令将会写入到这个纹理中。这样所有渲染操作的结果将会被存储在一个纹理图像上，可以对其处理实现各种效果。

## 渲染缓冲对象附件
渲染缓冲对象（renderbuffer Object）是在纹理之后引入到OpneGL中，作为一个可用的帧缓冲附件类型。优点是会将数据存储为OpenGL的原生渲染格式，是为离屏渲染到帧缓冲优化过的。

渲染缓冲对象直接将所有的渲染数据储存到它的缓冲中，不会做任何针对纹理格式的转换，让它变为一个更快的可写储存介质。然而，渲染缓冲对象通常都是只写的，所以你不能读取它们。

因为它的数据已经是原生的格式了，当写入或者复制它的数据到其它缓冲中时是非常快的。所以，交换缓冲这样的操作在使用渲染缓冲对象时会非常快。

## 渲染到纹理
```c++
...
 //帧缓冲
unsigned int framebuffer;
glGenFramebuffers(1, &framebuffer);
glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);
// 生成纹理
unsigned int textureColorbuffer;
glGenTextures(1, &textureColorbuffer);
glBindTexture(GL_TEXTURE_2D, textureColorbuffer);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB,GL_UNSIGNED_BYTE, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
// 将它附加到当前绑定的帧缓冲对象
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0,GL_TEXTURE_2D, textureColorbuffer, 0);
...
while(!closewindow){
    ...
    glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);	    //启用帧缓冲

    glEnable(GL_DEPTH_TEST);		                    //深度测试
    glClearColor(0.1f, 0.1f, 0.1f, 1.0f);		        //背景颜色	
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFER_BIT);	//清除颜色、深度缓冲
	...
	
	glBindFramebuffer(GL_FRAMEBUFFER, 0);	            //结束帧缓冲
	glDisable(GL_DEPTH_TEST);		                    //结束深度测试
	glClearColor(1.0f, 1.0f, 1.0f, 1.0f);	            //帧缓冲中Texture颜色
	glClear(GL_COLOR_BUFFER_BIT);		                //清除帧缓冲  
	
	//渲染纹理
	screen.use()
	glBindVertexArray(quadVAO);
    glBindTexture(GL_TEXTURE_2D, textureColorbuffer);
    glDrawArrays(GL_TRIANGLES, 0, 6);
    ...
}
```

## 一些后处理效果
```GLSL
//fragment
//反相
void main()
{
    FragColor = vec4(vec3(1.0 - texture(screenTexture, TexCoords)), 1.0);
}


//灰度
void main()
{
    FragColor = texture(screenTexture, TexCoords);
    float average = (FragColor.r + FragColor.g + FragColor.b) / 3.0;
    FragColor = vec4(average, average, average, 1.0);
}

void main()
{
    FragColor = texture(screenTexture, TexCoords);
    float average = 0.2126 * FragColor.r + 0.7152 * FragColor.g + 0.0722 * FragColor.b;
    FragColor = vec4(average, average, average, 1.0);
}


//核效果
const float offset = 1.0 / 300.0;  

void main()
{
    vec2 offsets[9] = vec2[](
        vec2(-offset,  offset), // 左上
        vec2( 0.0f,    offset), // 正上
        vec2( offset,  offset), // 右上
        vec2(-offset,  0.0f),   // 左
        vec2( 0.0f,    0.0f),   // 中
        vec2( offset,  0.0f),   // 右
        vec2(-offset, -offset), // 左下
        vec2( 0.0f,   -offset), // 正下
        vec2( offset, -offset)  // 右下
    );

    float kernel[9] = float[](
        -1, -1, -1,
        -1,  9, -1,
        -1, -1, -1
    );

    vec3 sampleTex[9];
    for(int i = 0; i < 9; i++)
    {
        sampleTex[i] = vec3(texture(screenTexture, TexCoords.st + offsets[i]));
    }
    vec3 col = vec3(0.0);
    for(int i = 0; i < 9; i++)
        col += sampleTex[i] * kernel[i];

    FragColor = vec4(col, 1.0);
}

//模糊
float kernel[9] = float[](
    1.0 / 16, 2.0 / 16, 1.0 / 16,
    2.0 / 16, 4.0 / 16, 2.0 / 16,
    1.0 / 16, 2.0 / 16, 1.0 / 16  
);

//边缘检测
float kernel[9] = float[](
    1, 1, 1,
   1, -8, 1,
    1, 1, 1  
);
```



