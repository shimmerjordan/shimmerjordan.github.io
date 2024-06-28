---
title: GAMES101 Lec5-6 光栅化notes
date: 2024-06-28 13:41:34
tags:
  - GAMES101
  - 渲染
categories:
  - 图形学
id: games101-lec5-6
---

# After MVP
> 在做了MVP把物体转化为一个Canonical cube之后，我们需要做什么？
> ——将其投影到屏幕

屏幕由像素点（可以暂时理解为一个小方块）构成，是离散的，二维的。像素由RGB的混合有了颜色

<!-- more -->

## Canonical Cube
先不考虑$z$的关联，需要通过*视口变换*将平面$\left[-1,\; 1\right]^2$转为$[0, weight] \times [0, height]$：$M_{viewport}=\left(\begin{array}{ccc} \frac{width}{2} & 0 & 0 & \frac{width}{2} \\ 0 & \frac{height}{2} & 0 & \frac{height}{2} \\ 0 & 0 & 1 & 0\\ 0 & 0 & 0 & 1 \end{array}\right)$，这样就可以在投影之后同时将左下角定义为$(0,\; 0)$

# 光栅
## 显示设备
- 示波器：通过偏移电子成像在感光平面；采用隔行扫描降低成像工作量，但容易出现画面撕裂
- 现代屏幕：将内存/显存的一部分区域显示到屏幕上
- 平板显示设备Flat Panel Displays：
	- LCD：Liquid Crystal Display通过调整自身排布影响光极化（偏正方向）
	- LED、OLED等等
	- Electronic Ink Display
# 光栅化
## 三角形
计算机生成图像中，最基本的二维元素是三角形，对其的离散化操作就是生成图像的重点。至于为什么选择三角形：
- 最基础的多边形，其他多边形可以划分成三角形
- 必定在一个平面；内外区分分明；插值方便（定义其中一个点的属性之后可以根据到三边距离等确定其他位置点的 属性）
我们需要将图像显示在屏幕上，例如下图我们怎么判断绿框中的位置的RGB（亮或不亮）呢：

![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240529172003566.png)

## 离散化-采样
>这就是为了解决刚才提到的问题，确切的说是判断一个像素和三角形的位置关系 or 像素中心点和三角形的位置关系
### 原理
Sampling实质上是将一个函数离散化的过程：一个像素点对应一个坐标点，对这个坐标点采样，判断它在不在三角形里面。

![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240529175700443.png)

- 可以采用上图的右图的蓝色包围盒的方式做光栅化的加速（不处理边界外的）
- 也可以用其他加速方法，例如左边的incremental triangle traversal（适用于窄长并且有旋转角度的，这样加速才明显）
# Antialiase 反走样（抗锯齿）

- Sampling artifacts（统称为瑕疵）
## 采样的缺点：
- 空间上Signals are changing too frequent (high frequency), but sampled too slowly
	- Jaggies (Staircase Pattern)：an example of “aliasing” – a sampling error
	- Moiré Patterns in Imaging 摩尔纹
- 时间上Signals are changing too fast (high frequency), but sampled too slowly
	- Wagon Wheel Illusion (False Motion)：倒着转的轮子
> Artifacts 背后的原因：信号变化过快，采样速率跟不上
## Antialiased Sampling

### 先Filter后采样
- Filter包括诸如卷积的方法：Getting rid of certain frequency contents 滤波器
	- Convolution (= Averaging) 卷积（可以看作一种低通滤波器）

![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240530113650706.png)
### 傅里叶变换
![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240530115904028.png)

$f(x)=\frac{A}{2}+\frac{2A\cos(t\omega)}{\pi}- \frac{2A\cos(3t\omega)}{3\pi}+ \frac{2A\cos(5t\omega)}{5\pi}-\dots$
另外注意到$e^{ix}=\cos x +i \sin x$，可以得到以下结论（傅里叶及逆变换）：

![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240530120445706.png)

这样就知道，任何一个函数都可以分解成不同频率函数的综合
### 采样频率
![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240530121608262.png)

sampling 是在频域上 copy & paste 波形
- Aliasing = Mixed Frequency Contents
- 采样率不够的 copy & paste 导致波形会有重叠（不恰当但是恰到“不好”处的采样频率会在不同的函数中采样出相同的结果）

# Reduce Aliasing Error
## Option 1: Increase sampling rate
- increasing the distance between replicas in the Fourier domain
- Higher resolution displays, sensors, framebuffers
- costly & may need very high resolution
## Option 2: Antialiasing
- Filtering out high frequencies before sampling
- Antialiasing = Limiting, then repeating
### Antialiased Sampling：Pre-Filter → Sample
> convolving = filtering = averaging

- 先采样后模糊（Filter）是不行的（波形重叠的情况下截断依然会有重叠）
- 时域下的卷积 = 频域下的乘积，时域下的乘积 = 频域下的卷积
![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240530144755652.png)

### Antialiasing By Supersampling (MSAA)

最理想的状态：我们需要一个近似的方法完成反走样，但不实际真的从硬件上增加很多像素点/分辨率。这就是MSAA的存在动机
- 近似方法
- Monte-Carlo
- 像素的**颜色值**为负责的区域内取样多次颜色值的平均。
![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240530151452898.png)
### MSAA的缺陷
No free lunch!
- MSAA：每个像素多次采样，求平均。太浪费性能
- 优化：不使用均匀分布，采样**复用**
- 怎样分布样本才能达到最好的覆盖效果：Blue Noise?
## Antialiasing Today

### Milestones：目前得到广泛应用
- FXAA(Fast Approximate AA)：先获得有锯齿的图，再**后处理**去除锯齿（很快）
	- 找到边界，换成没有锯齿的边界，（图像匹配）非常快
	- 方法和采样无关，采样虽然有误，但是这种方法可以弥补
- TAA (Tem'poral AA) ：时序信息，借助前面帧的信息
	- 最近刚刚兴起
	- 静态场景，相邻两帧同一像素用不同的位置来sample
	- 把MSAA的Sampling分布在时间上
	- 运动情况下怎么办？
- Super resolution / super sampling 超分辨率
    - From low resolution to high resolution
    - Essentially still “not enough samples” problem 类似抗锯齿
    - DLSS (Deep Learning Super Sampling) 猜
# Visibility / Occlusion
## 深度缓存 Z-buffer
**Painter's Algorithm**：由远及近画画，覆盖。（油画是这样的思路）
- 深度计算与排序：$O(nlogn)$
- 可能有无法排序的情况：例如三个三角形互相重叠
![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240530165233467.png)

**因此引入Z-buffer**：对每个像素多存一个深度
![Edge Image Viewer (gitee.com)](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Lec5-6-%E5%85%89%E6%A0%85%E5%8C%96/image-20240530165630546.png)
- 实际coding中，(smaller z -> further, larger z -> closer)
```C++
for (each triangle T)
	for (each sample (x, y, z) in T)
		if (z < zbuffer[x, y])            // closest sample so far
			framebuffer[x, y] = rgb;      // update color
			zbuffer[x, y] = z;            // update depth
		else
			;             // do nothing, this sample is occluded
```
- 复杂度：$O(n)$ for n triangles --> 并不是排序，而是只要**最值**
- 需要保证三角形进入顺序和结果无关
- 无法处理透明物体