---
title: GAMES101 Lec10-12 几何 notes
date: 2024-07-22 13:41:34
tags:
  - GAMES101
  - 渲染
categories:
  - 图形学
id: games101-lec10-12
mathjax: true
---

# 几何的表达方式

## Implicit 隐式几何
- Based on **classifying points**，例如通用一点的话：$f(x,y,z)=0$
- Sampling Can Be Hard
- Inside/Outside Tests Easy

<!-- more -->

### 种类
- Algebraic surface
	![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703163749790.png)
	- 最直接的，用数学公式
	- 不直观，复杂模型几乎不可能
- Constructive Solid Geometry（CSG）
	- 基本形状的布尔操作组合成复杂形状
	![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703163910473.png)
- Distance functions 距离函数
	- 相关：Signed Distance Field
	- [Ladybug (shadertoy.com)](https://www.shadertoy.com/view/4tByz3)这个例子用纯粹的距离函数表示了很nice的效果（注意这个渲染过程会拉爆你的显卡）
	- 解析形式表达任意一点到物体的最小距离
	- Blending 距离函数，模拟一个交融的过程
	![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703165134240.png)
- Level sets （水平集）
	- Grid方式描述distance，可以参考等高线？但其实就是距离函数的另一种表现方式
	![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703165841025.png)
	- 例子：CT扫描，上面的例子是二位的，这里3D的CT就和前面说到的纹理联系起来了
- Fractals 分形
	![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703170229826.png)
	- 自相似
	- 递归
- ...

### Pros
- compact description (e.g., a function)
- 很容易判断某点的位置certain queries easy (inside object, distance to surface)
- good for ray-to-surface intersection 光线求交比较简单 (more later)
- for simple shapes, exact description / no sampling error
-  easy to handle changes in topology (e.g., fluid)

### Cons:
- 很难描述，当然了也就很难去model complex shapes

## Explicit 显式几何
- All points are given directly or via parameter mapping: $f:\mathbb{R}^{2}\rightarrow \mathbb{R}^{3};(u,v) \Rightarrow (x,y,z)$，例如这样的一个马鞍面
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703163137053.png)
- Sampling Is Easy
- Inside/Outside Test Hard

### 种类
- Point cloud
	- 一堆点，一个$(x,y,z)$的列表
	- 可以表示任何几何，只需要点足够密
	- Useful for LARGE datasets (>>1 point/pixel)
	- Often converted into polygon mesh
	- Difficult to draw in undersampled regions
- Triangle/polygon mesh
	- Store vertices & polygons (often triangles or quads)
	- More complicated data structures
	- Perhaps most common representation in graphics
	- 例子：.obj格式
		- Just a text file that 物理点 specifies vertices, 法线 normals, 纹理坐标 texture coordinates and their connectivities
- subdivision, NURBS
- Bézier surfaces 贝塞尔曲线
- subdivision surfaces
- NURBS
- ...

No “Best” Representation, Each choice best suited to a different task/type of geometry
# 曲线 Curve
## Bézier Curves 贝塞尔曲线
一条由四个点（其实是任意≥3个点）定义的曲线：
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703173059441.png)
- p0和p3定义起点和终点
- p1和p2定义起点与终点的切线方向（与p0和p3一起）

### 怎么画一个贝塞尔曲线
>Evaluating Bézier Curves (de Casteljau Algorithm)

例子：三点来画曲线(quadratic Bezier)
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703173620214.png)
在$b_0b_1$和$b_1b_2$上分别找到对应时间$t$的相应比例的点$b_0^1$和$b_1^1$，再找到$b_0^1b_1^1$上对应比例的点$b_0^2$，这个$b_0^2$就是曲线上的一点，只要枚举所有的$t$就可以绘制出来曲线了。

四个点起始的话也是类似的，每次递归多计算一条边即可。

> 你可以在这个牛逼的网站[Hackery, Math & Design — Acko.net](https://acko.net/)找到这个牛逼的动画[Making things with Maths (acko.net)](https://acko.net/files/fullfrontal/fullfrontal/wdcode/online.html)（Youtube视频的16:00)

如果写出来的代数表达式就是：$$\left\{ \begin{aligned} b_0^{1}(t)=(1-t)b_0+tb_{1} \\ b_1^{1}(t)=(1-t)b_1+tb_{2}  \\ b_0^{2}(t)=(1-t)b_0^{1}+tb_{1}^{1} \end{aligned} \right. \Rightarrow b_{0}^{2}(t)=(1-t)^{2}b_0+2t(1-t)b_{1}+t^{2}b_2$$
更通用的，如果有$n+1$个控制点，我们可以得到一个$n$阶的Bézier曲线：
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703205522229.png)
其中Bernstein多项式为：$B_i^{n}(t)=\begin{pmatrix} n \\ i \end{pmatrix} t^{i}(1-t)^{n-i}$，其实就是描述多项分布的多项式：![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240703210139297.png)
我们通过定义这样一个与时间$t$有关的多项式来对不同的控制点进行插值，以此形成新的在曲线上的点。可以发现有以下性质：
- 过起点和终点：$b(0)=b_0$;  $b(1)=b_3$
- 对于三次（四个控制点）曲线：$b^{\prime}(0)=3(b_1-b_0)$; $b^{\prime}(1)=3(b_3-b_2)$
- 仿射变换前后统一（只需要变换控制点即可）；*注意例如投影变换的其他变换不一定满足*
- 凸包性质：形成的曲线一定在控制点形成的凸包内

但是高阶贝塞尔曲线很难控制，任何一个点就能影响全局，为了解决这个问题，我们引入了分段贝塞尔曲线。

### Piecewise Bézier Curves
- Demo：[Bezier Curve Edit](http://math.hws.edu/eck/cs424/notes2013/canvas/bezier.html)
- chain many low-order Bézier curve
- 例如常用的三阶：Piecewise cubic Bézier（每段四个控制点）
- 保证光滑（切线不突变）：内部控制点前后的切线点共线
	- $C^0$连续是无断点：$a_n=b_0$
	- $C^1$连是无突变（导数连续）：$a_n=b_0=\frac{1}{2}(a_{n-1}+b_1)$
	![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240709163022692.png)

## 其他曲线
1. Spline (样条)：a continuous curve constructed so as to pass through a given set of points and have a certain number of continuous derivatives. （一个可控曲线）
2. B-splines
	- basis splines 基函数样条
	- Bernstein Polynomial作为基函数
	- 是贝塞尔曲线的超集
	- 满足局部性（改动一个控制点，可以知道其影响范围而不是整条曲线）
	- 可能是图形学里面最复杂的一部分
3. Further：B样条、NURBS（非均匀有理B样条）[Prof. Shi-Min Hu’s course](https://www.bilibili.com/video/av66548502)

# 曲面
## Bézier Surfaces贝塞尔曲面
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240709164146390.png)
4x4个点：四条4个控制点的贝塞尔曲线，取同一时间（比如说$u$）获得四个控制点，取时间$v$，即获得最后的曲面上的点

### Evaluating Bézier Surfaces
总体思路：
1. 在时间$u$，计算出4条贝塞尔曲线，同时得到 “moving” 贝塞尔曲线的4个控制点
2. 在时间$v$，计算出 ”moving“ 贝塞尔曲线的表面点即为$(u,v)$
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240709164949623.png)

## Mesh
更广泛的还是Mesh（网格）
Mesh Operations: Geometry Processing
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240709165613584.png)
- Mesh subdivision 细分 upsampling
    - Increase resolution
- Mesh simplification 简化 downsampling
    - Decrease resolution
    - Try to preserve shape/appearance
- Mesh regularization 正规化
    - 不会出现特别奇怪的三角形（接近正三角形）
    - Modify sample distribution to improve quality

### 细分
#### Loop Subdivision
> **Loop是发明者名字，跟循环没关系**

细分的应用场景1：Displacement mapping 位移贴图 需要模型足够细致，于是需要细分（最好是动态细分）

需要三角形Mesh

步骤：

1. create more triangles (vertices)：Split each triangle into four
2. tune their positions （调整位置-形状需要有改变）
	- Assign new vertex positions according to weights
	- New / old vertices updated differently 新老点分别改变
		- 新的点：
			![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240709203222220.png)
			这里例子中中间共享的白色点可以更新为$\frac{3}{8} \cdot (A+B)+\frac{1}{8}\cdot (C+D)$，可以理解为是周围点的加权平均，使得其变得更加平滑
		- 旧的点：受自己本身和邻居的位置影响，
			![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240721221305224.png)
			$n$为顶点的vertex degree（度），旧点更新为$(1-n*u)*original\_postion+u*neighbor\_position\_sum$

#### Catmull-Clark细分
Loop细分只能处理三角形网格，对于更一般的网格，可以考虑Catmull-Clark细分。

**几个定义：**
- Non-quad face：非四边的面
- Extraordinary vertex (奇异点)：指(degree != 4)的点

**Each subdivision step：**
1. Add vertex in each face
2. Add midpoint on each edge
3. Connect all new vertices

简而言之就是连接这个面的中点和每条边的中点。需要注意的是，每一个非四边形面都会引入一个奇异点，然后这个非四边形面会消失（增加了非四边形面个数的奇异点，以后都不会增加了。也就是说所有非四边形面在第一次细分都会消失）

**另外，针对面上或者边上的新点，以及老的点各自的更新方法：**
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240722121849783.png)
虽然看起来很复杂，无非就是用以前的平均来定义新的update，类似模糊的平滑操作。

# Mesh Simplification
Goal: reduce number of mesh elements while maintaining the overall shape 
应用：移动端、远距离（LOD）

## Edge Collapse：顶点合并（边坍缩）
哪些边合并？如何合并？

### Quadric Error Metrics（⼆次误差度量）
- 放在二次误差之和最小的地方
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240722124513859.png)

仅仅简单求平均位置是显然不合理的，我们可以通过最小化一个点到原平面相关的**其他平面的距离和**（sum of L2 distance）来找到这个最合理的点

### Simplification via Quadric Error
- Garland & Heckbert 1997.
- iteratively collapse edge with smallest score
- 有问题：一条边的操作会影响其它边，需要更新
    - 数据结构：优先队列 or 堆（每次取最小后动态更新最小）
- 贪心算法，非全局最优（但也够用了）
- 可以有的放矢
	![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec10~12%20几何/image-20240722145336627.png)
