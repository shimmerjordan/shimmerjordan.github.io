---
title: GAMES101 Lec7-9 Shading notes
date: 2024-06-28 17:41:34
tags:
  - GAMES101
  - 渲染
categories:
  - 图形学
id: games101-lec7-9
mathjax: true
---
> 光照、着色、着色频率、图形管线、纹理映射

# 着色（光照与基本着色模型）
> 着色和阴影无关；
> Shading：The process of applying a material to an object.

<!-- more -->

## Blinn-Phong Reflectance Model 光照模型 着色模型
在考虑着色的时候不考虑物体遮挡导致的阴影
![image-20240625161511850.png (1482×699) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240625161511850.png)
### Difusse Relection 漫反射
表面颜色在任意角度是一样的，观测结果具有角度无关性
![image-20240625163240398.png (868×575) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240625163240398.png)
根据能量守恒，假设光能量是无损传播的，在直径为$1$的光球壳出的能量密度为$I$，那么在直径为$r$处的能量密度就是$I/r^2$。那么咱们可以又前面反射图所揭露的基本原理得出：$L_d=k_d(I/r^2)max(0,\mathbb{n} \cdot \mathbb{l})$，其中$L_d$是漫反射光线，$k_d$表示颜色的漫反射系数（吸收某种颜色能量的能力），$max(0,\mathbb{n} \cdot \mathbb{1})$表示shading point接收到的能量，因为小于0的物理意义是从物体内部反射出来的光线，没有现实意义因此与0做最大选择。
### Specular Term 高光
和漫反射不一样的点在于漫反射会向所有方向反射光线而高光部位只会向一个特定角度反射，因此高光部位的观测是角度相关的
**Blinn-Phong模型**
利用判断半程向量是否和法线“贴近”来判断观测方向是否在高光反射范围内：
![image-20240626163915819.png (1420×709) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240626163915819.png)
- 这个模型可以降低很多计算量，相较于根据入射方向和法线计算反射方向，半程向量的计算十分简单。
- $p$的作用：虽然说两个向量点乘可以判断彼此夹角大小，但$\cos{\alpha}$的容忍度太高，即使是有$45\degree$的夹角，也有很大的$\cos{\alpha}$值。以$p$次方来放大这种夹角敏感度。一般控制在100~200，控制高光的大小，可以参考下图
![image-20240626165130764.png (1221×765) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240626165130764.png)
### Ambient Term 环境光
-  近似看待成常数，用来保证不是全黑，上色&提亮：$L_a=k_{a}I_{a}$
- 但这仅仅是一种approximate，或者说是fake的，具体的涉及到全局光照的内容，比较复杂，后面再说

### 整体效果
将前面说的Ambient、Diffuse、Specular相加就是Blinn-Phong反射模型的最终结果：
$\begin{aligned} L &=L_{a}+L_{d}+L_{s} \\ &=k_{a} I_{a}+k_{d}\left(I / r^{2}\right) \max (0, \mathbf{n} \cdot \mathbf{l})+k_{s}\left(I / r^{2}\right) \max (0, \mathbf{n} \cdot \mathbf{h})^{p} \end{aligned}$
![image-20240626170018669.png (1223×380) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240626170018669.png)
# Shading Frequencies 着色频率
- Flat shading：每个**三角形**的法线是一样的，shading一次获得颜色值。但效果不会很好的，可以说是十分抽象了
- Gouraud Shading：每个**顶点**做一次Shading，中间对颜色值插值
- Phong Shading：
	 - 对法线值做插值，对每个**像素点**做Shading
	 - Not the Blinn-Phong Reﬂectance Model
![image-20240626175046538.png (1124×796) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240626175046538.png)
但我们可以看到随着几何模型越来越复杂（定义的vertices越多），Flat的方式似乎也并不会在肉眼上有什么效果差距，计算量还小

## 顶点法线的计算方法
相邻面的法线求“平均”，或许我们还可以给各个面根据其面积等因素赋予一个权值，使得这种“平均”更合理：
![image-20240627194718263.png (438×401) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627194718263.png)
$N_v=\frac{\sum_{i}N_{i}}{\Vert \sum_{i}N_{i}\Vert}$，其中$N_i$就已经是赋予了单位向量权值（长度）的面法线向量
- Barycentric interpolation：后面会介绍这种向量插值方式（记得normalization）
# Graphics Pipeline 实时渲染管线
一个三维场景到渲染出的图像效果的流程成为渲染管线：
![image-20240627200148405.png (1234×820) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627200148405.png)
- Vertex Processing
    - Model, View, Projection transforms
    - Shading, Texture mapping
    - Output: Vertex Stream
- Triangle Processing
    - Output: Triangle Stream
- Rasterization
    - Sampling
    - Output: Fragment Stream
- Fragment Processing
    - Z-Buffer Visibility Tests
    - Shading, Texture mapping 纹理映射
    - Output: Shaded Fragments
- Framebuffer Operations
    - Output: image (pixels)

Shader Programs
- Program vertex and fragment processing stages 自己编程顶点和像素的着色流程
- Describe operation on **a single vertex (or fragment)** 每个元素都执行一次——通用的执行逻辑
- vertex / fragment shader
- More:
	- Geometry Shader
	- Compute Shader
# 小结
## 案例
到现在的知识已经可以去学习API了（OpenGL, DirectX）
推荐：ShaderToy，只需要关注着色
Shader千变万化，例子：[Snail Shader Program（超高端例子）](https://www.shadertoy.com/view/ld3Gz2)

## 现状
**当下的实时渲染：**
- 100’s of thousands to millions of triangles in a scene
- Complex vertex and fragment shader computations
- High resolution (2-4 megapixel + supersampling)
- 30-60 frames per second (even higher for VR)，现在更是夸张

**Graphics Pipeline Implementation: GPUs**
- Specialized processors for executing graphics pipeline computations
- Heterogeneous, Multi-Core Processor
- 高度并行的，并行度远高于CPU，但每个核心的性能并不及CPU。这种分发式的简单作业更适合图形学的计算

# Texture Mapping 纹理映射
我们如何表现一个平面的纹理图案呢？
![image-20240627203649856.png (1160×756) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627203649856.png)
以这个漫反射公式为例，$L_d$并不只是可以表示漫反射系数，它可以表示此像素点任意基本属性
- 三维物体表面都是二维的
- 纹理：图，有弹性，可以映射到表面。*但事实上很难做到纹理的三角形完全吻合物理模型的三角形（无缝衔接），这是一块几何上很深入的研究领域叫做“参数化”*
	![image-20240627204141070.png (1234×842) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627204141070.png)
- 纹理坐标系：$(u,v)$，三角形每个顶点对应一个uv坐标，为了方便处理坐标均在0到1之内
- 一张纹理可以使用多次，类似贴瓷砖？
- 纹理本身设计得好的话，多次“贴瓷砖”可以无缝衔接→Tilable Texture
    - 一种方法：Wang Tiling
## Barycentric coordinates 重心坐标→插值
> 三角形的三个顶点有各自不同的属性，插值所做的是让三角形内部**平滑过渡**三个不同属性。这种插值可以对任意一种属性进行操作

三角形三个顶点的坐标和内部某点坐标的关系：$\begin{array}{r} (x, y)=\alpha A+\beta B+\gamma C \\ \alpha+\beta+\gamma=1 \end{array}$，这样我们就可以用$(\alpha, \beta, \gamma)$表示这一点，其中三数都非负。可以利用这个关系做其它属性的插值
我们可以利用面积比来计算系数：
$\begin{aligned} \alpha &=\frac{-\left(x-x_{B}\right)\left(y_{C}-y_{B}\right)+\left(y-y_{B}\right)\left(x_{C}-x_{B}\right)}{-\left(x_{A}-x_{B}\right)\left(y_{C}-y_{B}\right)+\left(y_{A}-y_{B}\right)\left(x_{C}-x_{B}\right)} \\ \beta &=\frac{-\left(x-x_{C}\right)\left(y_{A}-y_{C}\right)+\left(y-y_{C}\right)\left(x_{A}-x_{C}\right)}{-\left(x_{B}-x_{C}\right)\left(y_{A}-y_{C}\right)+\left(y_{B}-y_{C}\right)\left(x_{A}-x_{C}\right)} \\ \gamma &=1-\alpha-\beta \end{aligned}$
对于三角形重心，系数均为$1/3$。既然原理都知道，那就没必要非要记住这个复杂的公式。
但有个问题：**barycentric coordinates are not invariant under projection!** 投影前后的重心坐标可能会变化，所以需要在**对应时间**计算对应的重心坐标来做插值，不能随意复用！例如z-buffer：对投影三角形中某个点需要逆计算出其在三维的物理坐标做插值然后投影回二维（所以建议针对三维空间的物体先做三维空间的插值再投影）
## Texture queries : Applying Textures
像素在三角形内→计算对应uv→取纹理对应颜色值→设置
### 问题1: Texture Magnification 纹理太小怎么办 → 插值
> 小分辨率例如256×256的纹理映射到高分辨率的银幕例如4K，映射到的纹理会被拉大（非整数坐标）
- 纹理像素：texel
- 多个pixel映射到了同一个texel
- 解决：
    ![image-20240627215620170.png (1251×489) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627215620170.png)
    - Nearest
    - Bilinear
        - Bilinear 插值 lerp
        - 水平+竖直插值→双线性插值
        ![image-20240627220112334.png (1315×771) ](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627220112334.png)
        - 最近的四个点插值
    - Bicubic 双向三次插值
        - 周围16个点做三次插值
        - 运算量更大，结果更好
### 问题2: Texture Magnification 纹理太大怎么办
- 一个pixel对应了多个texel → 采样频率不足导致 摩尔纹+锯齿（走样）
	![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627221744484.png)
- 解决：
    - Supersampling
        - 太浪费！
        - Just need to get the average value within a range
            - Point Query vs. **(Avg.) Range Query**
    - Mipmap：Allowing (**fast, approx., square**) range queries
        - 每一次长宽各减半 D=0,1,2,...（有点像downsampling）
        - “Mip hierarchy”
        - overhead: 只比原Texture多占用1/3的存储
        - 怎么知道层数D？约为相邻pixel的映射uv之间的距离取2的对数（左侧为Screen Space，右侧为Texture Space）
        ![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627222733351.png)
        - 如果计算出来需要的D是整数，就很方便
        - 如果计算出来需要的D不是整数→Trilinear Interpolation三线性插值
            - 分别在floor(D)和ceil(D)上做Bilinear Interpolation取颜色值之后再插值
            ![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627224521093.png)
        - Limitation：Overblur
        ![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627224813613.png)
            - 不是方块查询
            ![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627225000474.png)
            - Solution：各向异性过滤
            ![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240627225020315.png)
    - 各向异性过滤 Anisotropic Filtering
        - Ripmaps and summed area tables
	        -  长/宽/长和宽 各减半
            - Can look up axis-aligned rectangular zones
            - 但斜着的区域仍然没有很好解决（EWA过滤解决）
	            - EWA filtering 椭圆取样：利用多次查询求平均值的方法来处理不规则区域
        - overhead：3
        - 多少x：压缩到多少x，显存足够的情况下开越高越好
# Application of Texture
## 各种贴图
- texture = memory + range query (filtering)
- General method to bring data to fragment calculations
## Environment lighting - Environment Map
- 环境光贴图
- 例子：Utah Teapot
	- 经典：Stanford Bunny，Cornell Box
- Spherical Environment Map - 把环境光记录在球体上
	![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628154301358.png)
	- 球心：世界中心
	- 一个问题：靠近两级的地方拉伸/扭曲，想象地球仪展开
	- 解决方法：Cube Map
- Cube Map：立方体表面，从球心到球面的投影向外
	![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628154503858.png)
	- 扭曲更少，但是Need dir->face computation，计算量更大
## Store microgeometry
- Textures can affect shading! → define height/normal → Bump / Normal Map
	- 两者类似，都可以以假乱真
	- 改变表面的法线
- Bump Mapping 凹凸贴图（法线贴图），不把几何形体变得很复杂的情况下可以应用一个相对复杂的纹理，就可以改变每个点的高度（使得其法线发生变化），从而制造一种fake的阴影凹凸效果
	- Bump Mapping的Texture记录了高度移动
	- 不改变几何信息
	- 逐像素扰动法线方向
	- 高度 offset 相对变化，从而改变法线方向
	- 计算法线方向：切线的垂直方向
	![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628160951313.png)
	1. 原本的法线$n(p)=(0,1)$竖直向上，在定义的凹凸贴图上可以先计算切线再求法线：新$p$点的切线（导数）为$dp = c \cdot [h(p+1) - h(p)]$，进一步得出法线（逆时针旋转$90\degree$）得到$n(p)=(-dp,1).normalized()$
	2. 三维空间中原始点$n(p)=(0,0,1)$，其切线：$\frac{dp}{du}= c_{1}\cdot [h(u+1)-h(u)]$，$\frac{dp}{dv}= c_{2}\cdot [h(v+1)-h(v)]$，再旋转后得到$n=(-\frac{dp}{du},-\frac{dp}{dv},1)$
		- 在这里我们可以将$p$点附近定义为一个*局部坐标系*，使得其法线方向始终是$(0,0,1)$，随后再将这个坐标变换到世界坐标中。
- Displacement mapping 位移贴图
	- 输入相同（Texture与Bump Mapping可共用）
	- 改真的改变了几何信息，对顶点做位移
	![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628162349087.png)
	- 相比上更逼真，要求模型足够细致，运算量更高
	- DirectX有Dynamic的插值法，对模型**按需**做插值，使得初始模型不用过于细致
## Procedural textures
- 3D Procedural Noise + Solid Modeling
	- 定义空间中任意点的颜色
	- 并没有实际生成纹理图，是定义一个空间噪声函数+其他处理（二值化等）→映射
	- 例子：Perlin Noise
- Provide Precomputed Shading
    - Ambient occlusion texture map
        ![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628163423782.png)
        - 计算好的环境光遮蔽贴图，一些被遮蔽的地方就不需要计算光影了
        - 空间换时间
- Solid modeling & Volume rendering
    - 三维渲染，例如核磁共振
# Shadow mapping（Lec12回头讲的）
> 之前提到的着色并没有考虑其他物体甚至自身的其他部分对其的影响，这是不对的

光栅化下对全局光线传输、阴影的处理十分麻烦。
- draw shadows using rasterization
- An Image-space Algorithm
    - 不需要场景的几何信息
    - 有走样现象，直到现在也是这样
    - 思想：the points NOT in shadow must be seen both **by the light** and **by the camera**
## 步骤
- Pass 1: Render from Light
    - Depth image from light source → shadow map 记录深度图
    ![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628165959826.png)
- Pass 2A: Render from Eye
    - Standard image (with depth) from eye
    ![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628170545489.png)
- Pass 2B: Project to light
    - Project visible points in eye view back to light source 投影看到的点到之前光源处的“虚拟相机”的成像上，对比记录的深度是否一致
        - visible to light source → color
        - blocked → shadow
感觉每个光源对每个静态场景有一个shadow map
## 问题
![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628171616277.png)
可以看到有一些“脏脏”的黑绿块
- 走样、分辨率
- 数值精度问题
    - Involves equality comparison of ﬂoating point depth values means issues of scale, bias, tolerance
- 只能点光源、[硬阴影](https://www.timeanddate.com/eclipse/umbra-shadow.html)
![gitee.com](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec7-9-Shading/image-20240628172223516.png)