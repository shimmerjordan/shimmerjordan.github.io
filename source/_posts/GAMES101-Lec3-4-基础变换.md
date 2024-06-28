---
title: GAMES101 Lec3-4 基础变换notes
date: 2024-06-28 12:41:34
tags:
  - GAMES101
  - 渲染
categories:
  - 图形学
id: games101-lec3-4
---

# 基础变换（仿射变换）：旋转、缩放、切变、平移
- 先后顺序很重要，需要仔细想
- 仿射变换=线性变换+平移
- 线性变换=旋转+缩放+切变

<!-- more --> 
# 齐次坐标系
n维的点/向量，用$[n+1 \times 1]$的列向量表示：$\begin{pmatrix} n+1 \\ n \\ \vdots \\ 1 \; or \; 0\end{pmatrix}$，其中最后一个维度0表示向量，1表示点。这样就可以保证向量的平移不变性，以及以下性质：

> Valid operation if w-coordinate of result is 1 or 0 
> - vector + vector = vector 
> - point – point = vector
> - point + vector = point 
> - point + point = mid point

最后一行引用了这个性质：$\begin{pmatrix} x \\ y \\ w\end{pmatrix}$ is the 2D point $\begin{pmatrix} x/w \\ y/w \\ 1\end{pmatrix}$
- 变换用$[n+1 \times n+1]$的矩阵左乘来表示，可以做矩阵的分解和压缩。变换顺序是从向量/点开始从右往左；
- 多个变换可以组合起来变成一个矩阵→加速计算
- 先做线性变换，再做平移变换
# Viewing (观测) Transformation
- View (视图) / Camera transformation
- Projection (投影) transformation
    - Orthographic (正交) projection (cuboid to “canonical” cube $[-1, 1]^3$)
    - Perspective (透视) projection (frustum to “canonical” cube)
        - First “squish” the frustum into a cuboid
        - Then Do orthographic projection
可以把任意一个旋转分解成x, y, z面的旋转：
$$\mathbf{R}(\mathbf{n}, \alpha)=\cos (\alpha) \mathbf{I}+(1-\cos (\alpha)) \mathbf{n} \mathbf{n}^{T}+\sin (\alpha)\left(\begin{array}{ccc} 0 & -n_{z} & n_{y} \\ n_{z} & 0 & -n_{x} \\ -n_{y} & n_{x} & 0 \end{array}\right)$$
其中$n$为过原点的旋转轴方向向量，$\alpha$为旋转角度
- 四元数：更多是为了表述旋转与旋转之间的插值
# MVP
## **Model** transformation (placing objects)
> 选址、摆Pose

世界坐标系下有很多Object，用一个变化矩阵把它们的顶点坐标从Local坐标系（相对）转换到世界Global坐标系（绝对）。这就是placing objects
## **View** transformation (placing camera)
> 摄像机机位调整——一般规整为原点

我们看到的画面由摄像机捕捉，摄像机参数决定了我们在屏幕上看到的东西，这一步可以将世界坐标系转换到摄像机坐标系。
这个变换一般是先平移到原点，再旋转对齐$z$轴再旋转对其$x$轴，旋转矩阵可以从逆矩阵（从对齐的情况逆旋转到当前位置）推导：
- 写作：$M_{view}=R_{view}T_{view}$
- 平移到原点的$T_{view}=\left(\begin{array}{ccc} 1 & 0 & 0 & -x_e \\ 0 & 1 & 0 & -y_e \\0 & 0 & 1 & -z_{e}\\ 0 & 0 & 0 & 1 \end{array}\right)$
- 在旋转阶段先考虑逆旋转：$R^{-1}_{view}=\left(\begin{array}{ccc} x_{\hat{g} \times \hat{t}} & x_t & x_{-g} & 0 \\ y_{\hat{g} \times \hat{t}} & y_t & y_{-g} & 0 \\ z_{\hat{g} \times \hat{t}} & z_t & z_{-g} & 0\\ 0 & 0 & 0 & 1 \end{array}\right)$
- 因为$R^{-1}_{view}$是正交矩阵，因此逆矩阵就是其转置，very easy
> 正交矩阵：
> 1. 必须是一个方阵，即$n$行$n$列；
> 2. 矩阵中的每一列若视作向量，则这些向量均两两相互垂直；
> 3. 矩阵中的每一列若视作向量，则这些向量的长度均为1；
## **Projection** transformation
> 快门成像

摄像机坐标系，视锥体，再规整一下
### Orthographic Projection
这个很简单，难度不大。通常情况，We want to map a cuboid $\left[l,\; r\right] \times [b,\; t] \times [f,\; n]$ to the “canonical (正则、规范、标准)” cube $[-1, 1]^{3}$:
![正交投影](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec3-4-%E5%9F%BA%E7%A1%80%E5%8F%98%E6%8D%A2/image-20240525232015628.png)
### Perspective Projection
参考下图，其实我们只需要将远平面(f)通过$M_{persp->ortho}$“挤压”成和近平面(n)一个尺寸即可转化为前面我们已知处理方式的正交投影即可。
>可以在n平面左侧想象存在一个点光源，这样就可以更方便理解为什么远平面更大

![cdn by https://gcore.jsdelivr.net](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec3-4-%E5%9F%BA%E7%A1%80%E5%8F%98%E6%8D%A2/image-20240529111149680.png)
根据相似三角形法则可以很轻松地找到这样的关系：$y^{\prime}=\frac{n}{z}y$  ,  $x^{\prime}=\frac{n}{z}x$
![cdn by https://gcore.jsdelivr.net](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec3-4-%E5%9F%BA%E7%A1%80%E5%8F%98%E6%8D%A2/image-20240529112017170.png)
进一步的，我们可以在齐次坐标系中得到这样的关系：
$\left(\begin{array}{c} x \\ y \\ z \\1 \end{array}\right) \Rightarrow \left(\begin{array}{c} \frac{nx}{z} \\ \frac{ny}{z} \\ unknown \\1 \end{array}\right) \overset{multi.\; by \;z}{==} \left(\begin{array}{c} nx \\ ny \\ still \; unknown \\ z \end{array}\right)$，这就可以得到$M_{persp->ortho}=\left(\begin{array}{ccc} n & 0 & 0 & 0 \\ 0 & n & 0 & 0 \\ ? & ? & ? & ?\\ 0 & 0 & 1 & 0 \end{array}\right)$
那么这个第三行怎么推导呢，可以利用：
- 近平面$z$轴始终为$n$：$\left(\begin{array}{c} x \\ y \\ n \\1 \end{array}\right) \Rightarrow \left(\begin{array}{c} x \\y\\ n \\1 \end{array}\right) \overset{multi.\; by \;n}{==} \left(\begin{array}{c} nx \\ ny \\ n^2 \\ n \end{array}\right)$，确定为$(0\; 0\; A\; B)\left(\begin{array}{c} x \\ y \\ n \\1 \end{array}\right)=n^2$这样前面为$0$的结构（与$x$, $y$无关），也即$An+B=n^2$
- 远平面中心点$\left(\begin{array}{c} 0 \\ 0 \\ f \\1 \end{array}\right) \Rightarrow \left(\begin{array}{c} 0 \\0 \\ f\\1 \end{array}\right)==\left(\begin{array}{c} 0 \\ 0 \\ f^2 \\ f \end{array}\right)$，确定$Af+B=f^2$
进一步可以得到$M_{persp->ortho}$了，其中第三行是$(0,\; 0,\; n+f,\; -nf)$，最终可以知道$M_{persp}=M_{ortho}M_{persp->ortho}$
#### fovY和aspect ratio
- fovY：field-of-view 垂直可视角度
- aspect ratio：宽高比
![cdn by https://gcore.jsdelivr.net](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec3-4-%E5%9F%BA%E7%A1%80%E5%8F%98%E6%8D%A2/image-20240529145252467.png)
再根据下图，可以比较容易地写出：$\tan{\frac{fovY}{2}}=\frac{t}{\left|n \right|}$，$aspect=\frac{r}{t}$
![cdn by https://gcore.jsdelivr.net](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec3-4-%E5%9F%BA%E7%A1%80%E5%8F%98%E6%8D%A2/image-20240529145812169.png)
这样一来，我们只需要定义一个垂直可视角度fovY可一个宽高比就可以定义出完整的视锥，其他的变量都可以表示出来

 
