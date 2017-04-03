---
layout: post
title:  借助 Adreno Profiler 在 Unity 中还原手游渲染效果
comments: true
category: graphic
---

在*Unity*中还原手游渲染效果，主要为了学习其渲染技巧和观察不同参数下的渲染效果。同时，在还原过程中寻找一种自动化的还原方式。

测试的游戏为*剑侠世界*，下面两张图是真机上截下来的人物界面图。

![1]({{ site.url }}/Blog/images/RE_Unity_AdrenoProfiler/1.png)
![2]({{ site.url }}/Blog/images/RE_Unity_AdrenoProfiler/2.png)

## 主要工具

还原主要依赖于高通提供的 [Adreno Profiler](https://developer.qualcomm.com/software/adreno-gpu-profiler) 工具。 该工具的 Scrubber GL 能够抓取游戏的当前一帧的渲染流程，可以获得当前帧的渲染流程（具体到每一个 Draw Call 及 Draw Call 对应的 Context)、所有 Shader 代码、所有贴图（包含用于 PostProcess 的中间贴图）。

但是通过 Adreno Profiler 获取的资源不能直接在 Unity 中使用，需要对这些资源做一些处理。如 Shader 需要转换为 Unity 中能用的代码、 Model 导出后需要处理为能导入 Unity 的 fbx 格式等。

有兴趣可以尝试一下 [SnapDragon Profiler](https://developer.qualcomm.com/software/snapdragon-profiler)，是高通钦定的 Adreno Profiler 继任者。不过好像不是很稳定，经常崩溃。

## 还原细节

### Shader
Adreno Profiler 提取出来的 Shader 代码是在手机真机上跑的 OpenGL ES 代码，分为 Vertex Shader 和 Fragment Shader。由于这些代码已经被优化过好几次，因此可读性很低。为了能在 Unity 中使用这些代码，目前是直接把他们拿过来，合并为一个符合 Unity GLSL 标准的 Shader 代码。

目前这部分工作基本实现了自动化。写了一个简单的转换器，对提取出的 Vertex Shader 和 Fragment Shader 做词法分析和手动简单的语法分析后，替换关键字再合并为一个 Shader 代码。

转换器处理输出的代码基本上可以在 Unity 里直接用，但也可能存在需要手动调整部分代码。如还原剑侠世界，需要加入翻转顶点 UV 坐标的代码。

同时，由于一个模型的渲染可能经过多个 Shader 的处理才能得到最终效果。要处理这部分逻辑观察渲染流程，来确定某模型的渲染涉及到哪几个 Draw Call。如*剑侠世界*中，人物的渲染经过 2 个 Pass，且光照信息不同。

### Model
Adreno Profiler 并没有提供所有 Model 的一键导出，只能逐个手动倒出（一次只能导出一个 Draw Call 所画的面片）。导出格式为 obj，但文件格式不是标准的 obj 格式，需要手动修改一些标记，如 *a_Position* 修改为 *v*。

写了一个小 Python 工具，可以做自动转换。

得到可用的 obj 以后，通过 *Blender* 等工具转换成 Unity 可用的 fbx 格式即可。

Adreno Profiler 有时候会无法导出完整的 obj 模型，可能导出一半会报错。

### Texture
材质可以从工具中一次性全部导出，但与材质相关的一些信息，如材质用于哪个模型、用于哪次 Draw Call 需要在工具内查看。

每一个材质都会有一个 ID。因此可以在工具内查看 Program 并结合每个 Draw Call 的 API Calls 信息，来确定某一个 Shader 内的某个参数对应的是哪个材质。

### Parameter
提取出 Shader，Model，Texture 以后，可以在 Unity 中还原出基本的画面。但是 Shader 的大部分参数需要手动设置，包括使用了哪些材质。这些参数需要手动在工具内查看（点击查看某次 Draw Call -> 查看 Program -> 查看 Uniform）。

### PostImageProcess
由于 PostProcess 涉及到 C# 代码，因此这部分基本上是对着工具逐步还原的体力活。后处理需要的 Shader 可以从工具里提取并自动处理成 Unity 可用的代码。但后处理的流程和步骤（Shader 使用的先后顺序、Texture 的前后输入输出关系）需要手写 c# 代码进行还原。

普通的后处理 Shader 代码比较简单，可以直接看懂后处理的 Shader 代码逻辑，然后手写还原效果。如 Bloom 效果，可以直接使用 Unity 自带的 asset 然后调整参数达到差不多的效果。


## 效果

下面两张是按原参数还原的效果。人物的贴图、轮廓高光和 Bloom 基本上都还原了。
<figure>
	<img src="{{site.url}}/Blog/images/RE_Unity_AdrenoProfiler/14912083500914.jpg" width="200" style="block:inline">
	<img src="{{site.url}}/Blog/images/RE_Unity_AdrenoProfiler/14912083803199.jpg" width="200" style="block:inline">
</figure>

下图是修改了身体部分边缘高光颜色的效果。雪白雪白，白的发光。

<figure>
	<img src="{{site.url}}/Blog/images/RE_Unity_AdrenoProfiler/14912085966393.jpg" width="200">
</figure>

最后放个用相同 Shader 渲染出来的 Bunny。

<figure>
	<img src="{{site.url}}/Blog/images/RE_Unity_AdrenoProfiler/14912087460743.jpg" width="400">
</figure>






