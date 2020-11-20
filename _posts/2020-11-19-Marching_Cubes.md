---
layout: post
title:  Marching Cube
comments: true
category: graphics
---

levelset 是流体模拟中用于表示 free surface 界面常见的方法。虽然其极具灵活性，但涉及渲染等工作时，还是需要从 levelset 中重建出表面 mesh。而 Marhcing Cube 算法便是常见的重建算法之一。

顾名思义，Marching Cube 将背景均匀网格（三维下由大小相同的 cube 构成）与 levelset 做相交。每一个发生交集的 cube 的边上，可以记录一组相交点。再利用相交点，在每一个 cube 内部重建出三角面片，导出即可获得 obj 格式的 mesh 文件。

下文默认以三维做例子，二维下的处理方法类似且相对来说会更为简单。首先讲相交点的计算，然后再介绍如何从一组交点生成三角面片。

## 相交点计算

我们不考虑 cube 的面与 levelset 的相交，只考虑 cube 边与 levelset 的相交，故相交获得是一组点（vertex）集，而不是边（edge）集。

一个边 $$(V_0, V_1)$$ 若和 levelset 有相交，则该边的两个顶点 $$V_1, V2$$ 应该位于 levelset 切割界面的两侧。我们假设 levelset 切割面为 $$\phi=0$$ 处，则 $$\phi(V_1) * \phi(V_2) <= 0$$。

## 面片生成

由于我们的目标是三角面片的生成，而不考虑后续对应的几何处理，因此只需要针对每一个 cube 进行三角面片生成即可，且每个三角面片的处理独立。

一个 cube 有 12 个边，虽然每个边都可能和 levelset 发生相交，对应有 $$2^{12}$$ 种相交组合的情况。但仅有部分相交组合是有可能存在的，如下图分别是一种六边形的相交面和三角形的相交面。

<figure>
    <img src="{{ site.url}}/Blog/images/Marching_Cubes/cube_cut_face.jpg"
    width="80%" style="margin-left:auto;margin-right:auto">
</figure>

即便如此，levelset 与 cube 还是有非常多的相交可能，从三角形一直到六边形都有可能。若要编码去识别各种不同的相交情况，不是很实际。虽然可以预先枚举获得一个映射表来达到运行时的快速识别，但生成这个映射表也不是件很容易的事情。

所以一般会将 cube 拆分成五个四面体（如下图），单独对每个四面体继续作相交。一个四面体与 levelset 相交仅有可能产生三角形和四边形，情况简单且易于识别。这种方法也被叫做 Marching Tetrahedra。

<figure>
    <img src="{{ site.url}}/Blog/images/Marching_Cubes/cube_split.jpg"
    width="65%" style="margin-left:auto;margin-right:auto">
</figure>