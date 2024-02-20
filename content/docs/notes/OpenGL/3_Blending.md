---
title: "7、混合"
weight: 8
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---
# 混合(Blending)
对于一些透明的纹理或图片，需要用到混合。

混合决定了如何将输出颜色与目标缓冲区中已有的颜色相结合
![](https://ynsource.oss-cn-beijing.aliyuncs.com/opengl/混合.png)

控制混合有三种方法：
- 启用glEnable(GL_BLEND)；禁用glDisable(GL_BLEND)
- glBlendFunc(src, dest)
  - src：指定如何计算RGBA源混合因子（默认为GL_ONE）
  - dest：指定如何计算RGBA目标混合因子（默认为GL_ZERO）
- glBlendEquation(mode)
  - mode：指定如何混合src和dest的颜色（默认为GL_FUNC_ADD）


假设：
```src = GL_SRC_ALPHA```，```dest = GL_ONE_MINUS_SRC_ALPHA```

对于两张图片{{< katex >}}RGBA_{src}{{< /katex >}}和{{< katex >}}RGBA_{dest}{{< /katex >}}中每个像素都有：
{{< katex display >}}
src = alpha\\
dest = 1-0\\

R = (r_{src} * src) + (r_{dest} * dest)\\
G = (g_{src} * src) + (g_{dest} * dest)\\
B = (b_{src} * src) + (b_{dest} * dest)\\
A = (a_{src} * src) + (a_{dest} * dest)
{{< /katex >}}


关于[glBelndFunc()](https://docs.gl/gl3/glBlendFunc)、[glBlendEquation()](https://docs.gl/gl3/glBlendEquation)