---
title: Dformer
date: 2025-09-29 16:36:22
tags: CV
---

RGB-D 学习（1/5）

<!--more-->

# Motivation:

此前的想法一般是：在一个 RGB 与训练的 back-bone 上 fine-tuning 出深度信息，然后去做任务。但是这样的问题：

1.  RGB-D 输入就是图像和深度的pair，和RGB预训练仅输入图像方式不一致，导致了表征便宜
2. 微调过程可能破坏了内部表征分布
3. 双骨干架构可能带来更高的计算开销

本质上，就是预训练阶段没有考虑深度信息。所以就有了 Dformer 的想法，也就是在预训练阶段引入深度的概念，

# architecture

![image-20250930164020456](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20250930164020456.png)

首先对于 RGB图像和对应的深度图，先通过并行的 STEM 层进行处理，每个 STEM 层实际上是两个卷积操作。卷完到了编码器，变为 $\{\frac{1}{4},\frac{1}{8},\frac{1}{16},\frac{1}{32}\}$ 尺寸然后来编码多尺度特征。接下来进行预训练，最后丢进 decoder 里得出答案。

核心的block如下设计：
![image-20250930164900510](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20250930164900510.png)

分成两个模块： GAA 模块负责从全局视角提供目标定位能力；LEA 用大卷积核卷积，细化 RGB 表示的细节。

GAA 模块里做的是全局的注意力，但是注意因为我们会下采样到固定的分辨率，具体来说就是对于 Q，我们使用一个池化层对 RGB 和 深度信息进行下采样后concat在一起，生成的 Q 就是 $k\times k\times C^d$ 尺寸，而 $K= Linear(X_i^{rgb}),V = Linear(X_i^{rgb})$。 这里其实我不太理解的点是， KV 都取的是 rgb 信息，这样是不是不太平衡？如果我取至少是concat完的呢？好先跳过脑内小剧场，最后得到是 
$$
X_{GAA} = UP(V ·  softmax(Q^T K/\sqrt{C^d}))
$$
接下来还有 LEA 模块，以捕捉更多的局部细节。这里的创新点是，对深度特征利用了一个 Dconv 进行采样，然后用一个点积把他和 RGB 的信息结合在一起。也就是
$$
X_{LEA} = DConv_{k\times k}(Linear(X_i^d)) · Linear(X_i^{rgb})
$$
同时我们保留了一个基础模块，这个计算的方式和 LEA 相同：
$$
X_{base} = DConv_{k\times k}(Linear(X_i^{rgb})) · Linear(X_i^{rgb})
$$
接下来，他提出了一个很神秘的 RGB-D 预训练法，他首先先对 Imagenet 做了一个深度估计，生成一大堆图像-深度的对。接下来在编码器的顶部加一个分类头，构建分类网络：也就是最后一层的 RGB_out 会展平然后输入到分类头。这部分有点 confusing,读一下代码：

## Dformer v2

其实 Dformer v2 最关键的就是提出了一个几何自注意力，也就是利用深度得到空间先验信息然后引导自注意力分配，而不是再为深度单独建编码器/做特征融合。这样能在不显式编码深度的情况下，更高效地利用几何信息做RGB-D语义分割。

对图像patch求**深度距离矩阵** $D$ 与平面曼哈顿距离矩阵 $S$，再用两组可学习的“权重”做加权融合，得到 $G\in\mathbb{R}^{HW\times HW}$。这里做一个记忆化的权重训练，接下来就是几何注意力公式，在标准注意力上引入几何衰减：
$$
\mathrm{GeoAttn}(Q,K,V,G)=(\mathrm{Softmax}(QK^\top)\ \odot\ \beta^{G})V
$$

其中 $\beta\in(0,1)$；$G$ 取元素作幂得到 $\beta^G\in(0,1]^{HW\times HW}$，对角为1，**距离越远权重越小**。

每个注意力头使用不同的 $\beta$ 增强几何引导的多样性。

将全局 $G$ 分解为**列向先验** $G_y\in\mathbb{R}^{HW\times H}$ 与**行向先验** $G_x\in\mathbb{R}^{HW\times W}$，分别做
$\mathrm{GeoAttn}_y=(\mathrm{Softmax}(Q_yK_y^\top)\odot\beta^{G_y})$、
$\mathrm{GeoAttn}_x=(\mathrm{Softmax}(Q_xK_x^\top)\odot\beta^{G_x})$，
四阶段编码器（1/4、1/8、1/16、1/32）：**不显式编码深度**；只需把深度图做平均池化到各尺度，为每个GSA模块生成对应尺度的 $G$。**保留全局视野**，但通过 $\beta^G$ 让远距离配对被软性抑制，更聚焦几何相关区域。在NYUv2/SUNRGBD/DeLiVER上达到或刷新SOTA；例如**DFormerv2-L：58.4% mIoU**，计算量明显低于对标方法。、

