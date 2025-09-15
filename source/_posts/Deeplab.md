---
title: Deeplab 系列学习笔记
date: 2025-09-11 16:04:49
tags: CV,分割
---





学长推荐（10/20）

<!--more-->

 # V1

backbone：VGG 16

实际上做了三件事情：

1. 去掉了全连接层，但是并不是用卷积层替代
2. 去掉了两个池化层，这里的考虑是，如果我们做分割的话每个像素是很重要的，但是 pooling 的话实际上会模糊每个像素的位置，和我们分割对像素敏感的需求相悖。
3. 使用空洞卷积，因为我们去掉了池化层感受野变小了，所以就要用空洞卷积来扩大感受野。

# V2

backbone：ResNet-101 with dilated conv

V2 提出了一个很有意思的架构：ASPP（空洞空间金字塔池化）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18e86573e4ea90d184eb8363f063adb0.png#pic_center)

这里很好理解，因为空洞卷积不同的 dilition_rate 就可以对应不同尺寸感受野，而至于四个结果怎么结合在一起也非常简单：直接用 1*1 卷积卷起来就可以啦。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c45da8e61859b38042992304a19d5ecc.png#pic_center)

# V3

几点贡献：

1. 改进了 ASPP，具体来说就是在空洞卷积后使用 BN，作者说对训练很有帮助。

2. 增加了逐点卷积核 image pooling,这是为了解决空洞卷积带来的问题，因为当 rate 增大的时候一次空洞卷积覆盖到的有效像素会减小到 1，所以我们使用 1*1 的卷积，减少参数个数。另外pooling是用于提取全局特征。

3. 加深网络。实际上就是利用空洞卷积不断加深网络的一种思路，因为随着越来越深特征层越来越小，所以此时我们用空洞卷积就可以让他降低的没那么快（

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/687000f0200a259c9b01c745d4eba145.png#pic_center)

   
