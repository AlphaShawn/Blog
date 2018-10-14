---
layout: post
title:  Rendering - About Material
comments: true
category: graphic
---

前几天跟着 Games Webinar 看了[闰令琪老师的 Rendering Tutorial 分享](https://v.qq.com/x/page/e07406zte1e.html)。因为自己本科毕设题目和毛发渲染相关，而近几年顶会中的毛发渲染文章基本都出自令琪老师。所以之前看到 Webinar 安排了这次的分享，就非常期待。

我也以此为契机，重启一下博客写作。这段时间内，会先跟着 Games Webinar Rendering Tutorial 系列，写一些听后感、收获和个人理解。

令琪老师的讲座前半部分相对偏基础，但思路清晰，从材质到渲染，再到真实感，过渡衔接都很自然。后半部分介绍了目前前沿的材质方面的研究方向，很多都还是我之前没有接触过新概念、新思路。

## 什么是材质？

**材质决定了物体外貌，材质描述了光线与物体之间的作用关系**

当光线照射到物体表面的时候，会发生散射现象，包含反射和折射。其中反射分为漫反射和镜面反射。同时，光线还会在场景内发生多次散射，最终又照射在物体表面，这些光线叫做环境光。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/B9AF9FA1-172B-46C9-907A-68BC7F2EDCB3.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>


在分析之前，给出一些术语定义。如下图所示，光线方向由向量 $$\vec{l}$$ 表示，指向光源方向。$$\vec{n}$$ 为平面法向，$$\vec{v}$$ 为观察（相机）方向。$$\vec{h} = \frac{\vec{l}+\vec{v}}{2}$$ 被称为 half vector，在后面小节会说明他的作用。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/15395218057920.jpg"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>

### 漫反射
物体都会发生一定程度的漫反射，即使其表面非常光滑（如金属）。这是由于现实世界中没有绝对光滑的表面，在一定的观察尺度下（放大后观察），都存在凹凸不平的微表面结构。如下图所示，微观尺度下，是存在很不规则的反射。不过这里有一个假设，在微观尺度下，各个平面的反射是镜面反射，即出射光线和入射光线关于微表面的法线对称。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/15395221095808.jpg"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>


在图形学渲染计算中，通常将漫反射项称为 *Diffuse Term*。那么如何来计算这项的大小呢？一个简单的建模就是用入射光 $$\vec{l}$$ 与物体表面法线 $$\vec{n}$$ 之间的夹角余弦值来表示。即当入射光线与平面法线夹角的为 0 时（入射光线与平面垂直），漫反射最强，随着夹角增大，漫反射变弱。如下面式子所示：

$$
L_d = k_d * (I / r^2) * max(0, n*l)
$$

上式的第三项就是余弦值，为了避免计算中出现负值，因此取了一个 max 把负角度都归成 0。上式中的第一项 $$k_d$$ 是和物体表面相关的属性，计算中通常预设为一个 RGB 值描述物体表面颜色。第二项 $$I/r^2$$ 描述了入射光强弱（单位面积）。如下图所示，从光源出发，单位圆上单位面积的强度为 $$I$$。那么当光传播到半径为 $$r$$ 的圆上时，根据能量守恒，单位面积强度应该变成 $$I/r^2$$。这也说明，光随传播距离的衰减是二次的。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/15395223895030.jpg"
    width="300" style="margin-left:auto;margin-right:auto">
</figure>


对于这个建模的理解，令琪老师讲座里给了一个例子：夏天和冬天为什么会有冷热差别？这不是由地球离太阳的远近变化造成的，而是由于太阳直射点的变化。我们夏天觉得热，是因为太阳光直射（与地面法线的夹角小），冬天觉得冷，是因为太阳光斜射（与地面法线的夹角大）。

如下图（取自令琪老师课件里的图），展现了不同 $$k_d$$ 取值对渲染效果的影响。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/BD54039F-31E4-402A-8874-13C5567CB0B0.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>

### 镜面反射

对于镜面反射，在渲染计算中，由 *Highlight Term* 来描述，在术语上通常又称为 *specular*。假设光线反射方向为 $$\vec{r}$$（和入射光$$\vec{l}$$关于表面法线 $$\vec{n}$$ 对称）。那么根据生活中的经验，很自然地就可以想到用 $$\vec{v}$$ 与 $$\vec{r}$$ 的夹角，来描述相机接收到镜面反射强弱。在实际计算中，$$\vec{l}, \vec{n}, \vec{v}$$ 都是已知向量，但是 $$\vec{r}$$ 需要通过一些计算得到。为了计算简便，会使用 *half vector $$\vec{h}$$* 和 $$\vec{n}$$ 的角度近似 $$\vec{r}$$ 与 $$\vec{v}$$ 的角度。$$\vec{h},\vec{n}$$之间的角度与 $$\vec{r},\vec{v}$$ 之间的角度实际上是差了一倍的，但它们在极值的取值上是一致的。因此 *Highlight* 可以用下式表示：

$$
\vec{h} = bisector(\vec{v}, \vec{l}) \\
L_s = k_s * (I/r^2) * (max(0, \vec{n} * \vec{h}))^p
$$

前两项和漫反射式子中一致。同时，会对$$\vec{n}, \vec{h}$$之间角度的余弦值做个幂运算。这是考虑到余弦函数太过于平滑，而实际生活中高光一般是一个比较小的区域。下图（取自令琪老师课件）表示了不同 $$p$$ 取值下的余弦值，以及其对应高光渲染的效果。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/AC5F9B8A-AAF1-464D-BAC3-A4F138AC7A25.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/44F377B6-5764-4213-9735-4B369F15E7B7.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>


### 环境光
光线漫反射和镜面反射通常被称为直接光照。而间接光照对物体材质表现也非常重要。即那些在场景内经过多次散射后才照射至表面的光线，对物体表现的视觉效果也十分重要。通常我们称这些光为环境光。环境光的建模十分复杂，因此在实时渲染计算中，一个最古老也是最简单的办法是使用下面这个式子计算环境光

$$
L_a = k_a * I_a
$$

可以看到，这个式子中环境光计算不仅和入射光线 $$\vec{l}$$ 无关，和观察方向 $$\vec{v}$$ 无关。这是一个非常简单的近似。除了这种近似之外，还有离线烘培，实用贴图的办法来近似计算环境光。

### Blinn-Phone
上面小节中提到的漫反射+镜面发射+环境光，共同构成了一个渲染模型，叫做 Blinn-Phone 模型。如下图（引自令琪老师课件），展现了不同项对最终渲染的效果影响。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/495673B2-1DCF-4908-8796-14C4AED17DCE.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>


# Shading（着色）
对材质的分析，都是针对特定物体提出一套渲染模型，来描述物体上的某一点，它在光的照射下，有怎么样反应，吸收多少光，反射出多少光。而 Shading （着色），则是基于渲染模型，在计算机中呈现出物体，是将材质应用到物体上的过程。

### Shading Method
因为在着色计算中，肯定是需要把模型拆分成一个个部分单独进行渲染计算的。而根据拆分粒度的不同，我们可以得到三种不同的着色方法。

第一种是按三角面片着色，也称为 *flat shading*。每一个三角面片，可以从其三个顶点中，计算得到三角面片的法向量，从而根据渲染模型，计算出该三角面片的颜色。

第二种是按顶点着色，也称为 *gourmand shading*。针对每一个顶点，我们利用其存储的法向量，根据渲染模型可以计算得到该顶点的颜色值。然后每一个三角形中任意一点的颜色值，都可以由三角形三个顶点的颜色插值而来。

第三种是按像素着色，也称为 *phong shading*。对于每一个三角形中的每一个像素，可以根据所在三角形的三个顶点插值得到该像素的法向量（及相关物理量如法向），进而对每一个像素进行渲染模型计算，得到该像素的颜色值。

三种着色方法的效果图如下（图引自课件）。实际上，可以想见，当模型本身表示足够精细时（三角面片很多），三种着色方法得到的渲染效果应该是一致的。在实际编程中，如编写 Unity shaderlab 着色器，按顶点着色即在 vertex shader 中计算渲染模型，按像素着色即在 fragment shader 中计算渲染模型，然后按三角面片着色，我感觉不常见，真要找个对应的话应该是在 geometry shader 中进行计算渲染模型。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/0114E158-8560-4458-B2BA-745C0CC0AEE0.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>


# 真实感材质

如这里开始，令琪老师的 Rendering Tutorial 就讲的比较略了，很多概念都是一带而过，主要点了一下思想，说明了一下来源。我也大概记录一下，以后有机会再逐渐给这些东西补坑。

### BRDF

**BRDF就是材质**。BRDF 全称为 Bi-directional Reflectance Distribution Function，他的定义如下：

$$
f_r(w_i, w_o) = f_r(\theta_i, \phi_i, \theta_o, \phi_o)
$$

其中 $w_i, w_o$ 分别是入射方向和出射方向，他们分别可以由两个参数 $\theta, \phi$ 表示。因此上面的 BRDF 是一个 4D 函数。

既然 BRDF 就是材质，那么他应该能够描述物体表面与光的交互关系。实际上，BRDF 函数的值就代表了入射光线与出射光线的能量比。如下图（引自课件），可以发现漫反射（Diffuse）对应 BRDF 就是一个常数函数，即对于给定入射光线，任意方向出射的光能量都是相同的。而 Glossy 表面的 BRDF 则略有区别，只有特定方向的出射光线才有能量，而且能量分布是呈瓣状（lobe）。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/DD48191C-F6FA-4CE7-8C01-8CF083D99B4E.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>




### Microfacet BRDF

在分析漫反射的时候，我们提到了微表面结构。一般来说，微表面的法线分布决定了物体宏观的材质特征。如下图，微表面法线分布比较集中，宏观下表面就比较光滑（高光强）。而微表面法线分布比较分散，宏观下表面就比较粗糙（漫反射强）。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/E9A366E6-9E7A-4AD5-BF0E-ACABB82C5854.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>

如下面微表面BRDF表达式中，$D(h)$ 描述的就是微表面法线的分布情况。

$$
f(w_i, w_o) = \frac{F(w_i, \vec{h})*G(w_i, w_o, \vec{h})*D(\vec{h})}{ 4(\vec{n}, w_i)(\vec{n}, w_o)}
$$

### Isotropic / Anisotropic BRDF

大部分 BRDF 可以归为两类。一类是各向同性，另一类叫做各向异性。各向同性就是前文中提到的漫反射等模型。各向异性指的是物体表面对特定方向光反应的现象，也就是物体表面的法向是具有方向性的，如下图（取自课件）就是典型的各向异性材质。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/1D3A64F6-AC55-4874-AF6C-5E66A4F30A20.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>


# Material Capture (BRDF measurement)

自然界中材质多种多样，想在数学上用解析式表示囊括所有材质并真实表示他们的物理特性是不可能的。因此工业界中，通常通过对物体的测量，获得特点材质的 BRDF。如下图（取自课件）就是测量工具，叫做Gonioreflectometer。它大概的工作原理就是，固定测试样本，然后再固定相机位置，之后移动光源照射方向，不断记录相机获得的反射数据值。一轮完成后，再改变相机位置，继续进行测量。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/1E35526B-0787-4523-A61A-882FC52F0182.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>



# Advanced Material

这部分的图全都是取自课件～主要介绍了真实渲染领域几个的研究方向。

### Detailed / Glinty material

细节、闪闪发光的材质。没有细节的材质，往往会让人感觉过于完美，而显得不真实。如下图，分别是微表面模型和考虑了细节的渲染效果。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/15395299192029.jpg"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>

在微表面模型中，微表面的法线分布函数 $D(h)$ 决定了材质细节。在一半的渲染中，使用的法线分布函数都过于平滑，如下图中左边的高斯分布。而现实世界中，微表面的法线分布可能是下图右边一样，是噪声非常大、很抖的函数。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/BB66E0BA-0793-49DC-A6A8-DDB968873637.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>


### Hair/Fur material

头发不是表面能够描述的物体，一般是用线、圆柱来描述头发。著名的模型有*Kayjiya-Kay* 和 *Marschner* 模型。我的毕设就是做头发的渲染，在这里再挖一个坑，之后再补一篇关于头发的材质文章。

### Participating Media 

在现实世界中，如雾（Fog）和云（Cloud）都属于这种材质。他们的特点在于光线在他们内部传播过程中，随时能发生吸收、散射（如下图）。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/945C97F2-94ED-4153-ACCC-668A1ECE068E.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>

### Translucent Material

半透明材质也属于特殊的 Participating Media。最常见的例子有玉石、水母、皮肤。如果要描述这种材质，就需要用到次表面散射（Subsurface Scattering），他是 BRDF 的超集，也被称作 BSSRDF。

$$
S(x_i, w_i, x_o, w_o)
$$

如上式，它不仅考虑光的入射和出射方向，还会考虑光入射和出射的位置。因此对材质的描述能力更强。他和 BRDF 的区别也可以由下图展现。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/15395306137764.jpg"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>

### Cloth

衣服的渲染也是一门学问。如下图，纤维（Fiber）合并卷在一起称为股（Ply），股再合并卷在一起称为纱（Yarn），最后再用纱纺织得到衣服。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/B887009A-B643-4CAA-95EC-51DD1C282F95.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>

最直接渲染衣服的办法[Sadegi et al. 2013]就是把它当作面，然后根据编织的模式设计 BRDF 函数渲染。更精细一点的呢把衣服分割成不同的区域，视作 participating media 进行渲染[Jakob et al.2010, Schroder et al.2011]。再细致一点可以把衣服视作一根根的线，用渲染头发的方法进行渲染[Kai Schroder]。很显然，后两种办法都非常耗时。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/945578CE-4DB0-4FBA-B9BC-83F951951B62.png"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>


### Granular Material

盐、糖、沙子堆，这些都成为颗粒状材质。他们不同的于普通的物体，近了看颗粒都有自己的表面，远了看颗粒就很难再看见。最近也有研究[Meng et al2015]如果直接一粒一粒进行描述。

### Procedural Material

动态生成材质。一般来说可以根据参数生成噪声函数，用于材质的描述。如下图山的渲染，仅仅使用了噪声函数生成，可见噪声函数的强大。噪声函数还可以用于水的渲染、木头的渲染等等。

<figure>
    <img src="{{ site.url}}/Blog/images/Rendering_About_Material/15395313267635.jpg"
    width="500" style="margin-left:auto;margin-right:auto">
</figure>



# 总结

材质的学问博大精深，渲染确实是很有玩也是非常有挑战的一个研究领域。虽然我现在暂时没有做渲染相关的东西，还是希望能够在业余时间蹭点料，学习学习。这篇文章虽然内容不是特别多，而且还是整理一个演讲，但我还是拖了将近一个多星期才最终搞搞完。自己的表达能力还是比较欠缺。还是希望能以此为起点，多写写东西，在学习之后能够输出点东西。

