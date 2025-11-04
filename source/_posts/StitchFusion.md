---
title: StitchFusion
date: 2025-11-4 13:38:59
tags: CV
---

# Motivation

作者分析了此前多种方法的特性：

- 基于映射的方法（如 CMX）把多模态特征映射到共享特征空间，这种方法参数量过大
- 基于提示的方法（如 GoPT）只支持两种模态的融合，并且会引入一定程度的偏差
- 基于交换的方法（如 SegFormer）在模态交换和融合中存在较高的复杂度
- 直接重新训练多模态分支（如 DFormer）需要大规模训练集和时间

所以作者提出了一种高效，避免引入模态偏差，适应任意模态组合的模块 MoA 和一个新架构 StitchFusion。

# Method

![image-20251104161257845](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20251104161257845.png)

其实这个模块的实现非常极简，他的想法是对于多种主干（如 ViT，ConvNext）我们把他看成 decoder 和融合器，然后把一个 MLP 直接当作 encoder。

为了让 VFM 可以当作融合器，作者引入了 MoA 作为信息拼接模块。对于两个模态 $i,j$，我们可以通过以下公式生成 MoA 信息：

在经过 DropPath 和 FFN 之后，再次经过一个 MoA 模块以融合信息。![image-20251104161855560](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20251104161855560.png)

关于 MoA 模块的设计，其实非常简单，借用 LoRA 的思想。整体想法就是 MLP 下采样 →  DROPOUT+GELU+MLP 处理 → DropPath →残差链接。而直接使用 MLP，作者也给出了解释：

> 我们仅专注于探索所提出的新型模态融合视角的作用。因此，本文仅采用了一种相对简单且实用的模态融合装置，即受[19]启发的简单低秩适配模块。至于新颖的模态适配器设计，我们暂时未予考虑，相关研究将在后续工作中展开。

其实就是因为角度比较新，从缝合两个模态的想法进行考虑，所以直接设计了简单的 adapter，实际上也可以用更加复杂的 adapter 架构。作者同时也给出了集中不太一样的 adapter 架构：比如多个模态共享一个权重的 adapter sMoA，实际上就是把上面这两个不同的 MoA 权重都改成同一个的 sMoA 权重等等。

![image-20251104162154550](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20251104162154550.png)

这个方法比较突出的亮点在于，参数量很小，很多时候以一半多的参数量达到了 CMNext等方法甚至更好的效果。但是对于计算量来说，相对又是更小一点。总体而言，StitchFusion能够有效平衡多模态性能与较低的计算成本。后续如果要再优化的话，也就是从 GFLOPs 这个角度进行优化了。因为有一个比较反常的现象是：StitchFusion过程会通过编码器处理所有模态数据。然而，在预训练编码器中仅使用RGB模态的模型，当模态数量增加时，其GFLOPs反而更低。![image-20251104163641827](C:\Users\B1359\AppData\Roaming\Typora\typora-user-images\image-20251104163641827.png)
