---
title: HRNet学习笔记
date: 2025-08-15 12:20:34
tags: [CV]
---

学长推荐（4/20）

以后吃 brunch 不要吃猪脚饭和热卤拌饭，到工位直接昏迷两小时了

<!--more-->

HRnet 是打通多个视觉任务的 backbone，虽然最开始是以人体姿态识别为开始，但是后来发现 HRnet 实际上是很通用的。

## Motivation

在人体姿态识别的任务里，经常会需要生成高分辨率的 heatmap.而在之前的工作里，都是通过先 downscaling，再 upsampling 这样的思路来进行的。总体来说都是这样让不同分辨率进行串联：
![img](https://pic2.zhimg.com/v2-4bb8823c72d5ad82c7f65b2b8b1fc8a1_1440w.jpg)

HRnet 的动机就是考虑让不同分辨率的 feature map 进行并联，并且添加交互：![img](https://pic1.zhimg.com/v2-b3cde396f1fb2541c975ee0a8853d344_1440w.jpg)

## Method

在并联的基础上，同时增加了一个 Fusion 的过程，具体来说是
![img](https://picx.zhimg.com/v2-b5ef50df1bba72b0725bbad5cd742aaf_1440w.jpg)

- 同分辨率的层直接复制
- 需要升分辨率的用双线性插值 + 1x1 卷积统一通道
- 降分辨率的使用带有步幅的 3x3 卷积
- 把三个分辨率相加融合。

如果有四个分支的化，HRnet 给出了几种方式：

![img](https://picx.zhimg.com/v2-d0153d654016077dda8829eb26204ebf_1440w.jpg)

1. 使用分辨率最高的特征图，适用于关键点检测或者图像分类（v1）
2. 把所有特征图融合，适用于语义分割（v2）
3. 使用特征金字塔，适用于目标检测网络(v2p)

通过以上方式，HRnet 在整个网络中都保留了高分辨率的表征，也就是图中 x1 的分支。

## Experiment

HRnet 在多个任务中都表现良好，证明了自己的潜力：

### 姿态识别

![img](https://pica.zhimg.com/v2-fce9f3cddcb3f3384ec3367d6a671986_1440w.jpg)

在参数和计算量不增加的情况下，要比其他同类网络效果好很多。

### 语义分割

![img](https://pic2.zhimg.com/v2-d46e3c7646d821760969a318c4cbcffb_1440w.jpg)

### 目标检测

![img](https://picx.zhimg.com/v2-667ab6c9a75a8b7de68f029a0cbbf111_1440w.jpg)

![img](https://picx.zhimg.com/v2-5c7cced412e5da25607842df7e61be15_1440w.jpg)

### 分类任务

![img](https://pic1.zhimg.com/v2-e2ed3e81c2c339337ce4a897817ad8ce_1440w.jpg)
