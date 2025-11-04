---
title: ShareCMP
date: 2025-10-31 19:38:59
tags: CV
---

# Motivation

由于 RGB-P 这个模态现在还缺乏研究，并且针对 RGB-P 这个模态来说，实际上有很多发掘的空间。

- 现有的 RGB-P 方法主要通过计算表征线偏振度和线偏振角来作为输入，但是缺点在于这样过度简化了这个模态，可能限制了作为模态的有效性

- 之前的对于 RGB-D/RGB-T 设计的方法都是双分支架构，但是这样的设计导致参数量巨大，RGB-P 的特性带来的是需要在资源受限的自主水下航行器里运行。所以需要设计更高效的架构

所以作者设计了一种共享双分支的 encoder，并且提出了一种偏振注意力模块，利用四个方向的偏振图像生成更丰富特性的表征，以及根据不同角度存在差异的原理，进一步提出了 CPALoss。同时新给了一个四个角度的偏振图像数据集:UPlight.

# Method

## Encoder

其主干为 segformer，同时也基于 CMX 设计成。这里直接借鉴了 CMX 里的 FRM 和 FFM 以及 Segformer 的整体架构。作者设计了一个对于 RGB 和 偏振模态统一编码器，而这个编码器采用了共享参数。

每个阶段的就由三个模块组成：OPembed，ESatten，MFFN组成（三个模块都来自 segformer 的实现）：这里的过程其实就和 segformer 是差不多的，但是他做的是两个分支分别过一遍 OPEmbed，以及 ESatten，MFFN 之后经过 CMX 里的 FRM & FFM 输出到下一层。注意这里的特别之处在于，除了 OPembed 之外的所有东西都是共享参数量的，这也让整个encoder变得高效。

## PGA

为了更好表征偏振数据的特性，加入了一个 PGA 模块，因为偏振相机实际上可以生成很多角度，这里取0，45，90，135四个角度来提取特征，想法是通过偏振相机的特性扩大特征的感受野。实际上四个角度的特征分别过一个 3x3 的卷积层之后 concat 到一起，之后也都是 4d 的通道信息。

接下来为了把通道的信息做进一步处理，我们首先对这个 4d 的通道特征做一个 DWconv（1x1 通道卷积 + 3x3 分组卷积 + 2 dilation，得到感受野 5x5，叠加前面的就是 7x7 了）那么这里得到的通道注意力特征 $Attn_P$，实际上实现了对有利偏振特征的自适应选择和增强。

最后也是把信息 shortcut 一下并且经过 PReLU 处理得到特征 $I_p = PReLU(DWConv(f_p + Attn_p * f_P))$

这里本质上是一个特征增强的工作，实际上是类似 SEnet 的那个想法，同样是从通道角度来进行注意力加权。

## CPALoss

考虑到模态特性，作者引入了一个 CPAloss 和相应的对于偏振模态专门设计的head：CPAAhead。

> 插入一下，讲一下偏振里面比较重要的几个指标：线偏振角（AoLP）、线偏振度（DoLP）、线偏振角正弦值（SAoLP）和线偏振角余弦值（CAoLP）。其中，我们设计的SAoLP和CAoLP基于三角函数原理，相当于对线偏振角的分解。这些偏振模式表征方法均源自斯托克斯矢量S = {S0, S1, S2, S3}，该矢量用于描述光的偏振状态。斯托克斯矢量S可通过式（11）计算得出，该公式基于光的正交分量之间的强度和相位关系，全面描述了光的偏振特性。
>
> ![image-20251103223050449](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20251103223050449.png)
>
> ![image-20251103223100441](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20251103223100441.png)

CPAAhead 主要就是用特征 $f_i$ 去估计 AoLP 和 DoLP，结构也是比较简单，和 segformer 里的 MLP 解码器其实类似，构建了两个 1x1 卷积层，然后分别上采样以统一不同尺度的特征，并使AoLP或DoLP的估计值与真值在尺寸上保持一致。直接训练一下：

![image-20251103223611529](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20251103223611529.png)

![image-20251103224232926](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20251103224232926.png)

这个loss只在34层用。
