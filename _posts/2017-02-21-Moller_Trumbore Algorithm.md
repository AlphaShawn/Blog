---
layout: post
title:  Möller–Trumbore Algorithm
comments: true
category: graphic
---

简单总结一下 射线(Ray) 与 三角形（Triangle) 的相交算法。Möller–Trumbore Algorithm 能够在不计算 三角形 所在面的信息, 求得 交点(interection)。

射线的向量表示形式 :

$$	\vec{r} = o + t\vec{d}$$

其中 $$\vec{d}$$ 为 射线方向(默认为单位向量), $$o$$ 为射线起始点。

三角形内任意点 $p$ 的表示方式 :

$$ p = (1-b_1-b_2)*p_0 + b_1 * p_1 + b_2 * p_2$$ 

其中 $$p_0, p_1, p_2$$ 为三角形三个顶点坐标, $$b_1, b_2$$ 为参数, 满足 $$0 \le b_1, 0 \le b_2$$ 且 $$b_1 + b_2 \le 1$$

由上即可得到求解 交点 的方程:

$$ \vec{r} = o + t\vec{d} =  (1-b_1-b_2)*p_0 + b_1 * p_1 + b_2 * p_2 $$ 

变换形式后, 得到:

$$ o - p_0 =  -t\vec{d} + b_1 * (p_1 - p_0) + b_2 * (p_2-p_0) $$ 

用矩阵表示为:
$$
\begin{bmatrix}
-\vec{d} & (p_1-p_0) & (p_2-p_0)
\end{bmatrix}
\begin{bmatrix}
t \\ b_1 \\ b_2
\end{bmatrix} = o-p_0$$

令 $$\vec{e_1} = p_1 - p_0, \vec{e_2} = p_2 - p_0, \vec{s} = o-p_0$$

依据 [Cramer's Rule](https://en.wikipedia.org/wiki/Cramer%27s_rule), 解方程得到:

$$
\begin{bmatrix}
t \\ b_1 \\ b_2
\end{bmatrix} = 
\frac{1}
{\begin{vmatrix}-\vec{d} & \vec{e_1} & \vec{e_2}\end{vmatrix}}*
\begin{bmatrix}
\begin{vmatrix}s & \vec{e1} & \vec{e2} \end{vmatrix}\\
\begin{vmatrix}-\vec{d} & s & \vec{e2} \end{vmatrix}\\
\begin{vmatrix}-\vec{d} & \vec{e1} & s \end{vmatrix}
\end{bmatrix}
$$

至此已经可以求的 交点坐标, 且 $t$ 即为 射线起点 到 交点的距离。在代码实现中, 我们还可以用一些技巧减少运算量提高效率。

依据
$$
\begin{vmatrix}
a & b & c
\end{vmatrix} = (a\times c) \cdot b
$$

得到:
$$
\begin{bmatrix}
t \\ b_1 \\ b_2
\end{bmatrix} = 
\frac{1}
{(-\vec{d}\times \vec{e_2}) \cdot \vec{e_1} }*
\begin{bmatrix}
(s\times \vec{e_2}) \cdot \vec{e_1}\\
(-\vec{d}\times \vec{e_2}) \cdot s\\
(-\vec{d}\times s) \cdot \vec{e_1}
\end{bmatrix}
$$

同时, 还可以根据以下两点进一步加速计算。

> 1. 交点 若在三角形内部, 则 $0 \le b_1, 0 \le b_2$ 且 $b_1 + b_2 \le 1$
> 2. 射线与三角形相交, 则 $ t \ge 0$

即利用符号, 减少计算的数量。






