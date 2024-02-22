---
title: "9、模板测试"
weight: 10
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

## 模板测试（Stencil testing）
和深度测试一样，模板测试也有模板缓冲（Stancil Buffer）,通常每个模板值是8位的，意味着每个像素能有256种不同的模板值，当某个片段有一个模板值的时候，就可以选择丢弃或保留该片段。

使用模板缓冲大概步骤如下：
- 启用模板缓冲的写入。
- 渲染物体，更新模板缓冲的内容。
- 禁用模板缓冲的写入。
- 渲染（其它）物体，这次根据模板缓冲的内容丢弃特定的片段。

在渲染流程中，先经过混合测试，接下来是模板测试，最后是深度测试。这些都是在经过片段着色器后完成。

模板测试可以实现很多效果，如：镜面效果、模板阴影等。

## 相关函数
glStencilFunc、glStencilOp、glStencilMask。

- glStencilFunc：control how to pass/fail test
- glStencilOp：control result of passing/failing a test
- glStencilMask：可以当成一个开关，但与开启/关闭模板测试不同，0x00表示无法修改模板缓冲区，0xff表示可以修改缓冲区

## 绘制边框
大致流程如下：
```c++
glEnable(GL_DEPTH_TEST);                            //启用深度测试
glEnable(GL_STENCIL_TEST);                          //启用模板测试
glStencilOp(GL_KEEP, GL_KEEP, GL_REFERENCE);        //三种情况如下， KEEP指保持stencil值，REFERENCE修改为glStencilFunc中ref的值
//stencil fail， stencil pass & depth fail, stencil pass & depth fail
...
...
while(!WindowClose) {
    ...
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);
    ...

    glStencilFunc(GL_ALWAYS, 1, 0xff);              //总是通过模板测试
    glStencilMask(0xff);                            //允许修改stencil buffer

    Shander.Use();                                  //启用Shader
    Object.Draw();                                  //绘制，也同时修改通过Depth test，的stencil buffer值

    glStencilMask(0x00);                            //不允许修改stencil buffer
    glStencilFunc(GL_NOTEQUAL, 1, 0xff);            //若stencil值不为1，测试通过，渲染通过depth test的颜色。注意这里stencil buffer是没变的
    glDisable(GL_DEPTH_TEST);                       //关闭深度测试意味着片段不会被舍弃，在哪都能看到

    Object.Scale();                                 //放大物体，也可以准备一个稍大一点的物体版本，这样更好，不过会占用内存。
    OtherShader.Use();                              //启用另一个Shader；Shader可以实现很多效果，不过这里渲染边框只需要一种颜色。
    Object.Draw();                                  //这时绘制出来的是边框，因为小一点的物体在stencil buffer中为1，大一点的这个会在为0的地方进行绘制，注意这里没有改变stencil buffer

    glStencilMask(0xFF);                            //允许修改
    glStencilFunc(GL_ALWAYS, 0, 0xFF);              //总是通过stencil test，修改通过depth test的模板缓冲区中的值，注意这里的值修改为0，可以认为啥也没干
    glEnable(GL_DEPTH_TEST);                        //启用深度测试
    ...
}
```
效果如下：
![](https://s2.loli.net/2024/02/22/bpZqVg5NaWP8k3D.png)
此时模板缓冲为：
![](https://s2.loli.net/2024/02/22/1nVjXTo6LBxgfSa.png)

