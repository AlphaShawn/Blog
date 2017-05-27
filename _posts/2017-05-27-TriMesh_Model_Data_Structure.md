---
layout: post
title:  存储三角网格模型的数据结构
comments: true
category: graphic
---

用数据结构表示一个多边形构成的模型很简单，顶点加上多边形的索引就可以。但如果要对模型做一些操作，比如 Subdivision、Mesh Simplification，就需要记录模型更多的信息，以便进行操作。

这篇文章算是学习总结，如果以后有碰到其他更好的数据结构，也会更新在这篇文章里。

下文列出的几种表示方法有一定的相似之处，不同的表示方法支持的模型操作会有不同。一般来说，记录的冗余信息越多，模型操作的能力越强。首先给出一个最普通的模型数据结构。

## 索引三角网格

{% highlight c++ %}
class Model {
    vector<Vertex> vertices;
    vector<Face> faces;
    // other field
};

class Vertex {
    Point p;
    // other field
};

class Face {
    int v[3];
    // other field
};
{% endhighlight %}

模型记录所有的点信息。在记录面的信息时，使用索引而不是具体的点来表示。虽然简单，但是记录了一个模型的所有信息，应该是冗余最少的一种表示方式。但仅依靠这种表示方法，很难快捷（常数时间复杂度内）的获得一个点的相邻面、一个面的相邻面这种拓扑信息，因此它不适合在模型操作中使用。

## Data Structure Used in Subdivision Surfaces

这种表示方法主要用来支持模型的曲面细分，参考自*PBRT 3.7 节*。先放一个 C++ 表示的代码（做了部分简化）。

{% highlight c++ %}
class Model {
    vector<Vertex *> vertices;
    vector<Face *> faces;
    // other field
};

class Vertex {
    Point p;
    Face *startFace;
    // other field
};

class Face {
    Vertex *v[3];
    Face *f[3];
    // other field
};

class Line {
    Vertex *v[2];
    Face *f[2];
    int f0edgeNum;
};
{% endhighlight %}

可以发现，Model 和第一种表示方法很相似，只记录了点和面的信息。但差别在于，Vertex 和 Face 记录一些额外的信息。

同时注意到，Line 并没有记录到模型内，它主要作为辅助的数据结构，帮助构建整个模型（从第一种表示方法转换到这种表示方法）。

#### Face

Face 的具体含义如下图。*v[3]* 记录指向三个顶点的指针，以逆时针排列。*f[3]* 记录指向相邻三个面的指针，逆时针排列。

同时，*f[i]* 是 由顶点 *v[i]* 和 *v[(i+1)%3]* 构成边的**对应面**，如 *f[0]* 在和边 *v[0]-v[1]* 对应。

<figure>
	<img src="{{ site.url }}/Blog/images/TriMesh_Model_Data_Structure/14902379904369/14958716223238.jpg" width="400">
</figure>

#### Vertex

Vertex 除了记录自身坐标，还记录了与该点相邻的一个面的指针。有了额外的面信息，结合 Face 记录的信息，就可以实现对**该点相邻面**的遍历操作。

先定义两个宏，用来获得索引 i 的前驱和后继。

{% highlight c++ %}
#define NEXT(i) (((i)+1)%3) 
#define PREV(i) (((i)+2)%3) // simplification of (i - 1 + 3) % 3
{% endhighlight %}

假设一个顶点 V，它在 V.startFace (图中为 f ) 中的索引为 i，如下图。

<figure>
	<img src="{{ site.url }}/Blog/images/TriMesh_Model_Data_Structure/14902379904369/14958721668277.jpg" width="400">
</figure>

那么我们可以定义

{% highlight c++ %}
Face *prevFace(int i) {
    return f[PREV(i)];
}

Face *nextFace(int i) {
    return f[i];
}
{% endhighlight %}

在获得 *nextFace* 之后，我们找到顶点 V 在 *nextFace* 里的索引，以此可以找到下下一个面。不断迭代，我们就实现了相邻面的遍历。

#### Line

之前提到过 Line 主要是构建这种数据结构的辅助类。这个表示方法的核心是面信息的构建，而面信息中 **相邻面** 的初始化需要借助边。

构建的主要方法如下：

1. 维护一个 Line Set
2. 遍历面，假设为面 A，同时尝试把它的边加入到 Line Set 中
    1. 如果 Line Set 中已经有这条边，说明另外一个拥有这个边的面 B 添加过。且面 B 和 面 A 彼此是关于该边的对应面，借助边记录的信息，更新这两个面的数据。最后从 Line Set 中移除该边。
    2. 如果 Line Set 没有该边，初始化一个边，并记录当前边的索引（起点 V 的索引），加入到 Line Set 中。

具体的实现细节，参见 *PBRT 3.7.1 节* 。


## Winged-Edge Model

上一个表示方法中，整个数据结构的核心在于面，面记录的额外数据，串起了模型的拓扑信息，便于一些模型操作的实现。而 Winged-Edge Model，看名字就可以想见，它的核心在于边。

如下图，就是 Winged-Edge Model 中 边 记录的一些信息 (下图也是这种数据结构的名字出处，像一个蝴蝶翅膀) 。

<figure>
	<img src="{{ site.url }}/Blog/images/TriMesh_Model_Data_Structure/14902379904369/14958737510623.jpg" width="500">
</figure>

主要记录了如下一些信息：

* 起点 *aVertex*、终点 *bVertex*
* 从 *aVertex* 面向 *bVertex* 时，右侧的面 *aFace*，左侧的面 *bFace*
* 在 *aFace* 中，以该边为起点，顺时针 (CW) 下一条边 *aCWedge*，逆时针 (CC) 下一条边 *aCCWedge*
* 在 *bFace* 中，以该边为起点，顺时针 (CW) 下一条边 *bCWedge*，逆时针 (CC) 下一条边 *bCCWedge*


下面是数据结构的 C++ 代码

{% highlight c++ %}
class Model {
    list<Vertex *> vertexRing;
    list<Edge *> edgeRing;
    list<Face *> faceRing;
    // other field
};

class Vertex {
    Point p;
    list<Edge *> edgeRing;
    // other field
};

class Face {
    list<Edge *> edgeRing;
    // other field
};

class Edge {
    Edge *aCWEdge, *aCCWEdge;
    Edge *bCWEdge, *bCCWEdge;
    Vertex *aVertex, *bVertex;
    Face *aFace, *bFace;
    // other field
};
{% endhighlight %}

结构示意图如下。


<figure>
	<img src="{{ site.url }}/Blog/images/TriMesh_Model_Data_Structure/14902379904369/14958750703784.jpg" width="700">
</figure>

## 总结

1. 以上两种表示方法，都是很强大的数据结构，用来支持很多模型操作。但这些操作中，可能会修改模型的拓扑结构，这就需要我们反过来去维护、更新数据结构。比如下图，使用第二种数据结构表示，将一个面分裂成 4 个面，我们不仅要添加新的顶点和新的面，还要维护面的相邻关系。

    <figure>
        <img src="{{ site.url }}/Blog/images/TriMesh_Model_Data_Structure/14902379904369/14958757204022.jpg" width="700">
    </figure>

    维护数据结构，本身耗费一定的时间，也带来了一顶的编程复杂度。某种程度上讲，记录的冗余信息越多，维护、更新的代价越大。因此，在实际应用中，设计的数据结构在满足模型操作的同时，还要考虑到如何简化冗余信息，降低维护的代价。

2. 其实将后两个数据结构和第一种极简的数据结构做对比，他们其实利用了**空间换时间**的思想，通过记录冗余的信息，来加速对特定边、特定点访问、遍历。

## 参考资料

1. Physically Based Rendering : From Theory to Implementation, 3.7 [Link](http://www.pbrt.org)

2. Maintaining Winged-Edge Model, [Link](http://www.sciencedirect.com/science/article/pii/B9780080507545500487)

